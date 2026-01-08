# lxml로 Webスクレイピング

[![Bright Data Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/)

이 가이드는 Python에서 `lxml` 패키지를 사용하여 정적 및 동적 콘텐츠를 파싱하고, 일반적인 과제를 극복하며, 데이터 추출 프로세스를 간소화하는 방법을 설명합니다.

- [Python에서 lxml로 Webスクレイピング 사용하기](#using-lxml-for-web-scraping-in-python)
- [사전 준비 사항](#prerequisites)
- [정적 HTML 콘텐츠 파싱](#parsing-static-html-content)
- [동적 HTML 콘텐츠 파싱](#parsing-dynamic-html-content)
- [Bright Data プロキシ와 함께 lxml 사용하기](#using-lxml-with-bright-data-proxy)

## Using lxml for Web Scraping in Python

웹에서 구조적이고 계층적인 데이터는 HTML과 XML의 두 가지 형식으로 표현될 수 있습니다:

- **XML**은 사전 구축된 태그와 스타일이 없는 기본 구조입니다. 개발자가 자체 태그를 정의하여 구조를 만듭니다. 태그의 주요 목적은 서로 다른 시스템 간에 이해될 수 있는 표준 데이터 구조를 만드는 것입니다.
- **HTML**은 미리 정의된 태그를 갖는 웹 마크업 언어입니다. 이러한 태그에는 `<h1>` 태그의 `font-size` 또는 `<img />` 태그의 `display`와 같은 일부 스타일 속성이 포함됩니다. HTML의 주요 기능은 웹 페이지를 효과적으로 구조화하는 것입니다.

lxml은 HTML과 XML 문서 모두에서 작동합니다.

### Prerequisites

lxml로 Webスクレイピング을 시작하기 전에, 머신에 몇 가지 라이브러리를 설치해야 합니다:

```sh
pip install lxml requests cssselect
```

이 명령은 다음을 설치합니다:

- XML 및 HTML을 파싱하기 위한 `lxml`
- 웹 페이지를 가져오기 위한 `requests`
- CSS 선택자를 사용하여 HTML 요소를 추출하는 `cssselect`

### Parsing Static HTML Content

スクレイピング할 수 있는 웹 콘텐츠는 크게 정적과 동적의 두 가지 유형이 있습니다. 정적 콘텐츠는 웹 페이지가 처음 로드될 때 HTML 문서에 포함되어 있어 スクレイピング이 쉽습니다. 반면 동적 콘텐츠는 초기 페이지 로드 이후 JavaScript에 의해 지속적으로 로드되거나 트리거됩니다.

먼저, 브라우저의 **Dev Tools**를 사용하여 관련 HTML 요소를 식별합니다. 웹 페이지에서 마우스 오른쪽 버튼을 클릭하고 **Inspect** 옵션을 선택하거나 Chrome에서 **F12**를 눌러 **Dev Tools**를 엽니다.

![DevTools in Chrome](https://github.com/luminati-io/web-scraping-with-lxml/blob/main/images/DevTools-in-Chrome-1024x576.png)

화면 오른쪽에는 페이지 렌더링을 담당하는 코드가 표시됩니다. 각 책의 데이터를 처리하는 특정 HTML 요소를 찾으려면 hover-to-select 옵션(화면 좌측 상단의 화살표)을 사용하여 코드를 탐색합니다:

![Hover-to-select option in Dev Tools](https://github.com/luminati-io/web-scraping-with-lxml/blob/main/images/Hover-to-select-option-in-Dev-Tools-1024x587.png)

**Dev Tools**에서 다음 코드 스니펫을 확인할 수 있습니다:

```html
<article class="product_pod">
<!-- code omitted -->
<h3><a href="catalogue/a-light-in-the-attic_1000/index.html" title="A Light in the Attic">A Light in the ...</a></h3>
            <div class="product_price">
        <p class="price_color">£51.77</p>
<!-- code omitted -->
            </div>
    </article>
```

`static_scrape.py`라는 새 파일을 만들고 다음 코드를 입력합니다:

```python
import requests
from lxml import html
import json

URL = "https://books.toscrape.com/"

content = requests.get(URL).text
```

다음으로, HTML을 파싱하고 데이터를 추출합니다:

```python
parsed = html.fromstring(content)
all_books = parsed.xpath('//article[@class="product_pod"]')
books = []
```

이 코드는 `html.fromstring(content)`를 사용하여 `parsed` 변수를 초기화하며, 이는 HTML 콘텐츠를 계층적 트리 구조로 파싱합니다. `all_books` 변수는 XPath 선택자를 사용하여 웹 페이지에서 class가 `product_pod`인 모든 `<article>` 태그를 가져옵니다. 이 문법은 XPath 표현식에 대해 특히 유효합니다.

다음으로, 책 목록을 반복하면서 제목과 가격을 추출합니다:

```python
for book in all_books:
    book_title = book.xpath('.//h3/a/@title')
    price = book.cssselect("p.price_color")[0].text_content()
    books.append({"title": book_title, "price": price})

```

`book_title` 변수는 `<h3>` 태그 내부의 `<a>` 태그에서 `title` 속성을 가져오는 XPath 선택자를 사용하여 정의됩니다. XPath 표현식 시작의 점(`.`)은 기본 시작점이 아니라 `<article>` 태그부터 검색을 시작하도록 지정합니다. 

다음 줄에서는 `cssselect` 메서드를 사용하여 class가 `price_color`인 `<p>` 태그에서 가격을 추출합니다. `cssselect`는 리스트를 반환하므로 인덱싱(``[0]``)을 통해 첫 번째 요소에 접근하며, `text_content()`는 요소 내부의 텍스트를 가져옵니다. 

추출된 각 제목과 가격 쌍은 딕셔너리 형태로 `books` 리스트에 추가되며, 이는 JSON 파일에 쉽게 저장할 수 있습니다.

이제 추출한 데이터를 JSON 파일로 저장합니다:

```python
with open("books.json", "w", encoding="utf-8") as file:
    json.dump(books, file)
```

스크립트를 실행합니다:

```sh
python static_scrape.py
```

이 명령은 디렉터리에 다음 출력이 포함된 새 파일을 생성합니다:

![static_scrape.py JSON output](https://github.com/luminati-io/web-scraping-with-lxml/blob/main/images/static_scrape.py-JSON-output-1024x695.png)

이 스크립트의 전체 코드는 [GitHub](https://gist.github.com/vivekthedev/c1c5f0fb0e23cabfa3fa5c364b939f7c)에서 확인할 수 있습니다.

### Parsing Dynamic HTML Content

동적 콘텐츠를 スクレイピング하려면 [Selenium](https://www.selenium.dev/)을 설치합니다:

```sh
pip install selenium
```

YouTube는 JavaScript로 렌더링되는 콘텐츠의 훌륭한 예입니다. 키보드 입력을 에뮬레이션하여 페이지를 스크롤하는 방식으로, [freeCodeCamp.org YouTube channel](https://www.youtube.com/c/Freecodecamp)에서 상위 100개 비디오 데이터를 スクレイピング해 보겠습니다.

먼저 **Dev Tools**로 웹 페이지의 HTML 코드를 검사합니다:

![FreeCodeCamp page on YouTube](https://github.com/luminati-io/web-scraping-with-lxml/blob/main/images/FreeCodeCamp-page-on-YouTube-1024x576.png)

다음 코드는 비디오 제목과 링크를 표시하는 요소를 식별합니다:

```html
<a id="video-title-link" class="yt-simple-endpoint focus-on-expand style-scope ytd-rich-grid-media" href="/watch?v=i740xlsqxEM">
<yt-formatted-string id="video-title" class="style-scope ytd-rich-grid-media">GitHub Advanced Security Certification – Pass the Exam!
</yt-formatted-string></a>
```

비디오 제목은 ID가 `video-title`인 `yt-formatted-string` 태그 안에 있으며, 비디오 링크는 ID가 `video-title-link`인 `a` 태그의 `href` 속성에 있습니다.

`dynamic_scrape.py`를 만들고 필요한 모듈을 import합니다:

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys

from lxml import html

from time import sleep
import json

```

브라우저 드라이버를 정의합니다:

```python
URL = "https://www.youtube.com/@freecodecamp/videos"
videos = []
driver = webdriver.Chrome()

driver.get(URL)
sleep(3)
```

이전 스크립트와 마찬가지로, スクレイピング할 웹 URL을 담은 `URL` 변수를 선언하고 모든 데이터를 리스트로 저장할 `videos` 변수를 선언합니다. 

다음으로 브라우저와 상호작용하는 데 사용할 `driver` 변수(즉, `Chrome` 인스턴스)를 선언합니다. `get()` 함수는 브라우저 인스턴스를 열고 지정된 `URL`로 리クエスト를 보냅니다. 

그 후 웹 페이지의 모든 HTML 코드가 브라우저에 로드되었는지 확인하기 위해, 페이지의 어떤 요소에도 접근하기 전에 `sleep` 함수를 호출하여 3초 동안 대기합니다.

이제 더 많은 비디오를 로드하기 위해 아래로 스크롤을 에뮬레이션합니다:

```python
parent = driver.find_element(By.TAG_NAME, 'html')
for i in range(4):
    parent.send_keys(Keys.END)
    sleep(3)
```

`send_keys` 메서드는 `END` 키를 누르는 동작을 시뮬레이션하여 페이지 하단으로 스크롤하고, 추가 비디오 로드를 트리거합니다. 이 동작은 `for` 루프 내에서 4번 반복되어 충분한 비디오가 로드되도록 합니다. `sleep` 함수는 각 스크롤 후 3초 동안 일시 정지하여, 다시 스크롤하기 전에 비디오가 로드될 시간을 확보합니다.

다음으로, 비디오 제목과 링크를 추출합니다:

```python
html_data = html.fromstring(driver.page_source)

videos_html = html_data.cssselect("a#video-title-link")
for video in videos_html:
    title = video.text_content()
    link = "https://www.youtube.com" + video.get("href")

    videos.append( {"title": title, "link": link} )
```

이 코드에서는 driver의 `page_source` 속성에서 얻은 HTML 콘텐츠를 `fromstring` 메서드에 전달하여 HTML의 계층적 트리를 구성합니다. 

그런 다음 CSS 선택자를 사용하여 ID가 `video-title-link`인 모든 `<a>` 태그를 선택하며, 여기서 `#` 기호는 태그의 ID를 사용해 선택함을 의미합니다. 이 선택은 지정된 조건을 만족하는 요소 리스트를 반환합니다. 

이후 각 요소를 반복하여 제목과 링크를 추출합니다. `text_content` 메서드는 내부 텍스트(비디오 제목)를 가져오고, `get` 메서드는 `href` 속성 값(비디오 링크)을 가져옵니다. 

마지막으로 데이터는 `videos`라는 리스트에 저장됩니다.

이제 데이터를 JSON 파일로 저장하고 driver를 닫습니다:

```python
with open('videos.json', 'w') as file:
    json.dump(videos, file)
driver.close()
```

스크립트를 실행합니다:

```sh
python dynamic_scrape.py
```

스크립트를 실행하면 디렉터리에 `videos.json`이라는 새 파일이 생성됩니다:

![dynamic_scrape.py JSON output](https://github.com/luminati-io/web-scraping-with-lxml/blob/main/images/dynamic_scrape.py-JSON-output-1024x495.png)

이 스크립트의 전체 코드도 [GitHub](https://gist.github.com/vivekthedev/36489fbaf896eb7c06ebb9350dec298a)에서 확인할 수 있습니다.

### Using lxml with Bright Data Proxy

Webスクレイピング은 アンチボット 도구나 レート制限 같은 문제에 직면할 수 있습니다. プロキシ 서버는 사용자의 IPアドレス를 마스킹하여 도움을 줍니다. Bright Data는 신뢰할 수 있는 プロキシ 서비스를 제공합니다.

시작하려면 무료 체험에 가입하여 Bright Data에서 プロキシ를 얻습니다. Bright Data 계정을 생성한 후에는 다음 대시보드를 확인할 수 있습니다:

![Bright Data Dashboard](https://github.com/luminati-io/web-scraping-with-lxml/blob/main/images/Bright-Data-Dashboard-1024x461.png)

**My Zones** 옵션으로 이동하여 새로운 [residential proxy](https://brightdata.co.kr/proxy-types/residential-proxies)를 생성합니다. 그러면 다음 단계에서 필요한 プロキシ 사용자 이름, 비밀번호, 호스트가 표시됩니다.

다음으로 URL 변수 아래에 다음 코드를 추가하여 `static_scrape.py`를 수정합니다:

```python
URL = "https://books.toscrape.com/"

# new
username = ""
password = ""
hostname = ""

proxies = {
    "http": f"https://{username}:{password}@{hostname}",
    "https": f"https://{username}:{password}@{hostname}",
}

content = requests.get(URL, proxies=proxies).text
```

플레이스홀더를 Bright Data 자격 증명으로 교체하고 스크립트를 실행합니다:

```sh
python static_scrape.py
```

이 스크립트를 실행하면 이전 예시에서 받은 것과 유사한 출력을 확인할 수 있습니다.

이 전체 스크립트는 [GitHub](https://gist.github.com/vivekthedev/201f994bc14e4dbc7263b03983f917b3)에서 확인할 수 있습니다.

## Conclusion

Python에서 lxml을 사용하면 효율적인 Webスクレイピング이 가능하지만, 시간이 많이 소요될 수 있습니다. Bright Data는 바로 사용할 수 있는 [datasets](https://brightdata.co.kr/products/datasets)와 [Web Scraper API](https://brightdata.co.kr/products/web-scraper)를 통해 효율적인 대안을 제공합니다.

Bright Data를 무료로 사용해 보세요!