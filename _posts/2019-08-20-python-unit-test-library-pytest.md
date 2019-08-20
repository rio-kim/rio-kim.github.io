---
layout: post
title: Python unit test library pytest
categories: Python Testing
tags: test python unittest pytest
---
사내에서 프로젝트 진행 중 python flask로 간단한 rest api 서버를 구성할 일이 있었다. 규모가 점점 커지며 유닛테스트가 필요해져 pytest를 사용하여 유닛테스트를 진행했고 간단하게 세팅 및 사용기를 남긴다.

---
### pytest vs unittest
사실은 unittest가 파이썬 표준 테스팅 라이브러리이다. 그러나 구글에서 pytest vs unittest로 검색을 해보면 상당히 많은 포스트가 나오는데, 대부분은 pytest를 추천한다. 이유는 pytest가 unittest와 비교해 사용법이 더 편리하고, 파이썬 스타일 가이드(PEP8)를 따라 간결한 코드 작성에 도움이 되며 테스팅 프레임워크로써의 추가적인 장점(픽스쳐 활용, custom assert 활용)이 있기 때문이라 한다. 특히 django나 flask 등의 프레임워크 위에서 개발하고 있다면 pytest를 활용하는 것이 좋아 보인다.

각각의 비교 글들은 아래를 참고하세요.
- https://www.bangseongbeom.com/unittest-vs-pytest.html
- https://americanopeople.tistory.com/255
- https://www.reddit.com/r/Python/comments/5uxh22/unittest_vs_pytest/
- https://www.slant.co/versus/9148/9149/~unittest_vs_pytest

##### 1. pytest 사용을 위한 설정
우선 pytest 라이브러리 의존성 추가가 필요하다. pip를 통해 설치해준다. 경우에 따라서는 pytest-mock 등 pytest 기반 플러그인 라이브러리를 추가해야 하는 경우들이 생긴다.

인텔리제이의 경우 python test runner 설정을 하면 테스트 수행이 쉬워지고 결과도 인텔리제이 탭에서 확인할 수 있다. Tools > Python Integrated Tools에서 Default test runner를 pytest로 선택한다.

![check](https://user-images.githubusercontent.com/21053518/63341904-380db580-c385-11e9-9a13-157a9db72779.png)

##### 2. 테스트를 작성하기 전에 간단히 알아둬야 할 것들
- pytest에서 test suite으로 인식하는 파일명은 __prefix로 test_ 를 붙인다.__
- 마찬가지로 test 파일 내에서 생성하는 함수 이름도 __prefix로 test_ 를 붙여야 한다.__
- 테스트코드를 프로덕트 파일과 분리하여 놓을 필요는 없지만 가독성을 위해서 test 폴더 등을 만들고 그 밑에 몰아넣는 것이 좋다.

##### 3. 샘플 테스트 코드
가장 간단한 형태의 샘플 테스트 코드를 남긴다. contains_whitespace라는 메서드를 테스트하기 위한 코드이다. assert문을 이런식으로 사용한는 정도만 참고하고 자세한 assert문의 사용은 pytest docs를 열어보자.
~~~python
def test_contains_whitespace_with_valid_input():
    result = contains_whitespace('test')
    assert result is False

def test_contains_whitespace_with_invalid_input_all_whitespace():
    result = contains_whitespace('    ')
    assert result is True

def test_contains_whitespace_with_invalid_input_contains_whitespace():
    result = contains_whitespace('te  st')
    assert result is True
~~~

##### 4. 테스트 수행
터미널에서는
- __pytest test_code.py -k 'test_method_name'__ 명령으로 특정 메서드만 수행하거나
- __pytest test_code.py__ 명령으로 단건 파일을 수행하거나
- __pytest ./test_folder__ 명령으로 특정 폴더 내에 있는 테스트 코드 전체를 수행할수 있다.

인텔리제이에서는 테스트 파일을 우클릭하여 Run 'pytest ...' 또는 함수 옆에 생기는 초록색 플레이버튼을 통해 실행할 수 있다.

![image](https://user-images.githubusercontent.com/21053518/63343082-1c57de80-c388-11e9-8a20-7af244467ed0.png)

##### 5. mock의 사용
~~~python
def test_login_with_invalid_user_id(mocker):
    mock_response_json = json.dumps({
        'data': {
            'userId': 'test id',
            'password': 'test password'
        }
    })
    mock_response.text = mock_response_json
    mock_response.status_code = 400

    mocker.patch('requests.post', return_value=mock_response)

    result = login_service.login(user_id, password, login_ip)
    assert result['userId'] == 'test id'
    assert result['loginToken'] == 'test token'
~~~
샘플 코드는 requests.post 메서드를 mocker를 통해 mocking하고 login_service의 login 메서드를 테스트하기 위한 코드이다. login_service.login 메서드에는 requests.post 구문이 있을 것이고 그부분이 대체되어 실제로 호출이 되지 않고 return_value에서 정의한 값이 리턴되게끔 수행된다.

mocker를 쓰기 위해서는 우선 pytest-mock이라는 라이브러리가 필요하니 pip를 통해 설치해준다.
- 테스트 메서드에 파라미터로 __mocker__ 를 넣어준다.
- mocking할 대상이 클래스 없는 메서드인 경우는 __mocker.patch('패키지.파일.메서드명')__
- 클래스 내부의 메서드를 mocking할 때는 __mocker.patch.object(클래스, '메서드명')__
- __return_value__ 옵션을 활용하여 mock 메서드의 리턴값을 사용자가 정의할 수 있다.

##### 6. 익셉션 발생 여부 테스트
~~~python
def test_string_to_datetime_with_invalid_input():
    with pytest.raises(InvalidFormatException) as ex:
        string_to_datetime('some text')
    assert ex.value.invalid_fields['key'] == 'some text'
    assert ex.value.code == Fail.INVALID_PARAM
    assert ex.value.message == 'Invalid format: some text'
~~~

위 테스트코드는 InvalidFormatException이라는 익셉션이 발생하기를 기대하고 있다. __with 구문 밑에__ 테스트하고자하는 메서드를 수행시키고, 실제 발생된 익셉션 클래스는 __ex.value로 값 검증이 가능하다.__

~~~python
def test_string_to_datetime_with_valid_input():
    try:
        result = string_to_datetime('2019-01-01T09:00:00')
        assert result == datetime.datetime(2019, 1, 1, 9, 0, 0)
    except Exception as ex:
        pytest.fail(str(ex))
~~~
위의 테스트코드는 정상케이스 즉 익셉션이 발생하지 않는 경우에 대한 검증이다. try-except문으로 감싸서, 혹시 익셉션이 발생하는 경우에는 의도적으로 pytest.fail을 호출하는 식이다.

##### 7. fixture의 활용
~~~python
@pytest.fixture(scope="function")
def login_service():
    my_service = ...생략...
    return my_service

def test_code(login_service):
    result = login_service.login('test id', 'test password')
    ...생략...
~~~
pytest.fixture 구문을 이용하여 test에 필요한 fixture를 미리 준비할 수 있다. 사용하기 위해서 아래의 테스트코드와 같이 파라미터로 fixture 메서드명을 넣어주면 테스트코드 내에서 fixture를 쓸 수 있다.


##### 8. 정리
초반 설정만 대충 잘 해주면 간단한 테스트 코드는 금방 작성되고 쉽게 수행된다. mock의 사용도 어렵지 않다. assert문을 custom하게 만드는 부분은 아직 작성해 보지 않았으나 docs만 봐도 어려워 보이지는 않는다.

##### 9. 링크
- [pytest documentaion](https://docs.pytest.org/en/latest/contents.html)
- [pytest-mock(pypi)](https://pypi.org/project/pytest-mock)
