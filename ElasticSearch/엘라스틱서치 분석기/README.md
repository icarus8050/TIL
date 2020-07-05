# 엘라스틱서치 분석기

## 텍스트 분석 개요

 엘라스틱서치는 루씬을 기반으로 구축된 텍스트 기반 검색엔진입니다. 루씬은 내부적으로 다양한 분석기를 제공하는데, 엘라스틱서치는 루씬이 제공하는 분석기를 그대로 활용합니다.

 다음과 같은 문장이 있다고 해보겠습니다.

"프로그래밍하기 좋은 날씨, 즐거운 프로그래밍"

 일반적으로 특정 단어가 포함된 문서를 찾으려면 검색어로 찾을 단어를 입력하면 될 것이라 생각할 것입니다. 하지만 엘라스틱 서치는 텍스트를 처리하기 위해 기본적으로 분석기를 사용하기 때문에 생각하는대로 동작하지 않습니다. 예를 들어 "프로그래밍"이라는 단어를 입력하면 검색되지 않을 것입니다. 이는 "프로그래밍"이라는 단어가 존재하지 않기 때문에 해당 문서가 검색되지 않는 것입니다.

**엘라스틱서치는 문서를 색인하기 전에 해당 문서의 필드 타입이 무엇인지 확인하고 텍스트 타입이면 분석기를 이용해 이를 분석합니다. 텍스트가 분석되면 개별 텀(term)으로 나뉘어 형태소 형태로 분석됩니다. 해당 형태소는 특정 원칙에 의해 필터링되어 단어가 삭제되거나 추가, 수정되고 최종적으로 역색인 됩니다.**

 아까 위에서 예를 들었던 "프로그래밍하기 좋은 날씨, 즐거운 프로그래밍"이라는 문장이 실제로 어떻게 분석되는지 확인해 보겠습니다.

```
POST _analyze
{
  "analyzer": "standard",
  "text": "프로그래밍하기 좋은 날씨, 즐거운 프로그래밍"
}
```

 위의 분석 결과는 아래와 같이 token 값으로 표시됩니다.

```
{
  "tokens" : [
    {
      "token" : "프로그래밍하기",
      "start_offset" : 0,
      "end_offset" : 7,
      "type" : "<HANGUL>",
      "position" : 0
    },
    {
      "token" : "좋은",
      "start_offset" : 8,
      "end_offset" : 10,
      "type" : "<HANGUL>",
      "position" : 1
    },
    {
      "token" : "날씨",
      "start_offset" : 11,
      "end_offset" : 13,
      "type" : "<HANGUL>",
      "position" : 2
    },
    {
      "token" : "즐거운",
      "start_offset" : 15,
      "end_offset" : 18,
      "type" : "<HANGUL>",
      "position" : 3
    },
    {
      "token" : "프로그래밍",
      "start_offset" : 19,
      "end_offset" : 24,
      "type" : "<HANGUL>",
      "position" : 4
    }
  ]
}
```

 예제에서는 특별한 분석기 없이 Standard Analyzer를 사용했기 때문에 별도의 형태소 분석은 이루어지지 않았습니다. 텍스트를 분석할 때 별도의 분석기를 지정하지 않으면 기본적으로 Standard Analyzer가 사용됩니다.

## 역색인 구조

 어떤 책을 읽을 때 특정한 단어를 알고 있지만 해당 단어가 등장하는 페이지를 알지 못할 때 책의 마지막에 수록된 "찾아보기"에서 나열된 목록을 찾아보게 됩니다. 이 페이지에는 단어와 해당 단어가 포함된 페이지가 열거되어 있어서 특정 단어가 등장하는 페이지를 쉽게 찾아낼 수 있습니다. 루씬도 이와 비슷하게 동작합니다. 루씬의 색인은 역색인이라는 특수한 방식으로 구조화되어 있습니다.

 역색인 구조를 간단하게 정리하자면 다음과 같습니다.

-   모든 문서가 가지는 단어의 고유 단어 목록
-   해당 단어가 어떤 문서에 속해 있는지에 대한 정보
-   전체 문서에 각 단어가 몇 개 들어있는지에 대한 정보
-   하나의 문서에 단어가 몇 번씩 출현했는지에 대한 빈도

 예를 들어, 다음과 같은 텍스트를 가진 2개의 문서가 있다고 해보겠습니다.

문서1.

elasticsearch is cool

문서2.

Elasticsearch is great

 위의 두 문장에 대한 역색인을 만들기 위해 각 문서를 토큰화해야 합니다. 토큰화된 결과물은 대략적으로 아래와 같습니다. (실제로 저장되는 데이터는 훨씬 많은 정보를 저장합니다.)

<table style="border-collapse: collapse; width: 95.3488%; height: 202px;" border="1"><tbody><tr><td style="width: 198px;"><b>토큰</b></td><td style="width: 198px;"><b>문서번호</b></td><td style="width: 198px;"><b>텀의 위치 (Position)</b></td><td style="width: 197px;"><b>텀의 빈도 (Term Frequency)</b></td></tr><tr><td style="width: 198px;">elasticsearch</td><td style="width: 198px;">문서1</td><td style="width: 198px;">1</td><td style="width: 197px;">1</td></tr><tr><td style="width: 198px;">Elasticsearch</td><td style="width: 198px;">문서2</td><td style="width: 198px;">1</td><td style="width: 197px;">1</td></tr><tr><td style="width: 198px;">is</td><td style="width: 198px;">문서1, 문서2</td><td style="width: 198px;">2, 2</td><td style="width: 197px;">2</td></tr><tr><td style="width: 198px;">cool</td><td style="width: 198px;">문서1</td><td style="width: 198px;">3</td><td style="width: 197px;">1</td></tr><tr><td style="width: 198px;">great</td><td style="width: 198px;">문서2</td><td style="width: 198px;">3</td><td style="width: 197px;">1</td></tr></tbody></table>

 위 내용을 살펴보면 토큰이 어떤 문서의 어디에 위치하고, 토큰의 빈도수에 대한 정보를 알 수 있습니다. 이 결과를 바탕으로 검색어가 존재하는 문서를 찾기 위해 검색어와 동일한 토큰을 찾아 해당 토큰이 존재하는 문서 번호를 찾아가면 됩니다.

 하지만 여기서 "elasticsearch"를 검색어로 지정하면 우리가 예상했던 것과는 다른 결과가 나올 것입니다. 우리가 원하는 결과는 문서1, 문서2에 해당하는 내용이 모두 나와야하는 것입니다. 하지만 두 토큰(elasticsearch, Elasticsearch)은 첫 글자의 대소문자가 다르기 때문에 컴퓨터의 입장에서는 서로 다른 토큰으로 인식하여 문서1 만이 검색 결과로 나타나게 됩니다. 이러한 문제를 해결하기 위해서는 **어떻게 하면 해당 토큰들을 하나로 볼 것인가를 고민해야 합니다.**

이 문제의 가장 간단한 해결 방법은 색인 전에 텍스트 전체를 소문자로 변환한 다음 색인하는 것입니다. 그렇게 되면 두 개의 문서가 "elasticsearch"라는 토큰으로 나오게 될 것입니다.

<table style="border-collapse: collapse; width: 100%;" border="1"><tbody><tr><td style="width: 198px;"><b>토큰</b></td><td style="width: 198px;"><b>문서번호</b></td><td style="width: 198px;"><b>텀의 위치 (Position)</b></td><td style="width: 197px;"><b>텀의 빈도 (Term Frequency)</b></td></tr><tr><td style="width: 198px;">elasticsearch</td><td style="width: 198px;">문서1, 문서2</td><td style="width: 198px;">1, 1</td><td style="width: 197px;">2</td></tr><tr><td style="width: 198px;">is</td><td style="width: 198px;">문서1, 문서2</td><td style="width: 198px;">2, 2</td><td style="width: 197px;">2</td></tr><tr><td style="width: 198px;">cool</td><td style="width: 198px;">문서1</td><td style="width: 198px;">3</td><td style="width: 197px;">1</td></tr><tr><td style="width: 198px;">great</td><td style="width: 198px;">문서2</td><td style="width: 198px;">3</td><td style="width: 197px;">1</td></tr></tbody></table>

**색인한다는 것은 역색인 파일을 만드는 것입니다. 그렇다고 원문 자체를 변경한다는 의미는 아닙니다. 따라서 색인 파일들에 들어간 토큰만 변경되어 저장되고 실제 문서의 내용은 변함없이 저장됩니다. 색인할 때 특정한 규칙과 흐름에 의해 텍스트를 변경하는 과정을 분석(Analyze)이라고 하고 해당 처리는 분석기(Analyzer)라는 모듈을 조합해서 이루어집니다.**

---

## 참고자료

[엘라스틱서치 실무 가이드](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9791158391485&orderClick=LEa&Kc=) <<권택환, 김동우, 김흥래, 박진현, 최용호, 황희정 지음>>