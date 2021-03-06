---
title: Mockito를 사용하여 의존성들에 대해 mock 객체 생성하기
short-title: Mocking
prev:
  title: 단위 테스트 소개
  path: /docs/cookbook/testing/unit/introduction
next:
  title: Widget 테스트 소개
  path: /docs/cookbook/testing/widget/introduction
---

어떠한 경우에는 단위 테스트가 웹 서비스나 데이터베이스로부터 데이터를 가져오는 역할을
수행하는 특정 클래스에 의존하는 경우가 있습니다. 이럴 때는 다음과 같은 이유로 인해 
테스트가 어려워집니다:

  * 상용 서비스나 데이터베이스를 호출하는 것은 테스트 수행 성능을 저하시킵니다.
  * 성공해야 하는 테스트일지라도 만약 웹 서비스나 데이터베이스가 예기치 못한 결과를 
    반환하면 실패할 수 있습니다. 이러한 경우를 다른 말로 "flaky test"라고 합니다.
  * 상용 서비스나 데이터베이스를 사용하여 모든 가능한 성공과 실패 시나리오를 테스트하는
    것은 어렵습니다.

그러므로, 상용 웹 서비스나 데이터베이스에 의존하기 보다는, 이러한 의존성을 
"mock" 으로 대체할 수 있습니다. Mock은 상용 서비스나 데이터베이스를 흉내낼 수 
있으며, 상황에 따라 특정 결과 값을 반환하도록 할 수 있습니다.

Generally speaking, you can mock dependencies by creating an alternative
implementation of a class. Write these alternative implementations by
hand or make use of the [Mockito package][] as a shortcut.

본 예제에서는 Mockito 패키지를 사용하여 Mock 객체를 생성하는 기본적인 방법을 
아래와 같은 순서로 소개할 것입니다. 

  1. `mockito` & `test` 의존성을 추가하기.
  2. 테스트할 함수 생성하기.
  3. `http.Client` Mock 객체와 함께 테스트 파일 생성하기.
  4. 각 조건마다 테스트 작성하기.
  5. 테스트 수행하기.
  
더 자세한 내용은 [Mockito 패키지 문서]({{site.pub-pkg}}/mockito)를
참고하시기 바랍니다.

For more information, see the [Mockito package][] documentation.

`mockito` 패키지를 사용하기 위해, `pubspec.yaml` 
파일의 `dev_dependencies` 영역에
`flutter_test` 의존성과 함께 추가하세요.

본 예제에서는 `http` 패키지도 사용해야 하기 때문에, 
`dependencies` 영역에 추가해주세요.

```yaml
dependencies:
  http: <newest_version>
dev_dependencies:
  test: <newest_version>
  mockito: <newest_version>
```

## 2. 테스트할 함수 생성하기

In this example, unit test the `fetchPost` function from the
[Fetch data from the internet][] recipe.
To test this function, make two changes:

  1. 함수에 `http.Client`를 제공하세요. 상황에 따라 적당한 `http.Client`를 제공해줄 
     것입니다. Flutter와 서버-사이드 프로젝트에서는 `http.IOClient`를 제공할 수 있습니다.
     브라우져 앱의 경우는 `http.BrowserClient`를 제공할 수 있습니다.
     테스트 경우에는 `http.Client`의 mock 객체를 제공할 수 있습니다.
  2. mock 객체로 만들기 어려운 static `http.get()` 메서드가 아닌, 
     제공된 `client`를 사용하여 인터넷으로부터 데이터를 가져오세요

변경된 함수는 아래와 같습니다:

<!-- skip -->
```dart
class Post {
  dynamic data;
  Post.fromJson(this.data);
}

Future<Post> fetchPost(http.Client client) async {
  final response =
      await client.get('https://jsonplaceholder.typicode.com/posts/1');

  if (response.statusCode == 200) {
    // 만약 서버로의 요청이 성공했다면, JSON으로 파싱합니다.
    return Post.fromJson(json.decode(response.body));
  } else {
    // 만약 요청이 실패하게 되면, 에러를 던집니다.
    throw Exception('Failed to load post');
  }
}
```

## 3. `http.Client` Mock 객체와 함께 테스트 파일 생성하기

Next, create a test file along with a `MockClient` class.
Following the advice in the [Introduction to unit testing][] recipe,
create a file called `fetch_post_test.dart` in the root `test` folder.

`MockClient` 클래스는 `http.Client` 클래스를 구현할 것입니다. 이를 통해 
`MockClient`를 `fetchPost` 함수에 전달할 수 있게 되며, 각 테스트마다
다른 http 응답을 반환할 수 있게 해줍니다.

<!-- skip -->
```dart
// Mockito 패키지에서 제공되는 Mock 클래스를 사용하여 MockClient를 생성합니다.
// 각 테스트 마다 이 클래스의 새로운 인스턴스를 생성하세요.
class MockClient extends Mock implements http.Client {}

main() {
  // 테스트 코드는 여기에 위치합니다.
}
```

## 4. 각 조건마다 테스트 작성하기

`fetchPost()` 함수는 두 가지 일 중 하나를 수행하게 됩니다:

  1. http 요청이 성공하면, `Post` 를 반환합니다.
  2. http 요청이 실패하면, `Exception`을 던집니다.

그러므로, 이 두 가지 경우에 대해 테스트 코드를 작성해야 합니다.
`MockClient` 클래스를 사용하여 성공 케이스를 위해 "Ok" 
응답을 반환할 수 있고, 실패 케이스를 위해 error 응답을 반환할 수 있습니다.
이러한 조건을 Mockito가 제공하는 `when()` 함수를 사용하여 테스트하세요:

<!-- skip -->
```dart
// Mockito 패키지에서 제공되는 Mock 클래스를 사용하여 MockClient를 생성합니다.
// 각 테스트 마다 이 클래스의 새로운 인스턴스를 생성하세요.
class MockClient extends Mock implements http.Client {}

main() {
  group('fetchPost', () {
    test('returns a Post if the http call completes successfully', () async {
      final client = MockClient();

      // 제공된 http.Client를 호출했을 때, 성공적인 응답을 반환하기 위해 
      // Mockito를 사용합니다.
      when(client.get('https://jsonplaceholder.typicode.com/posts/1'))
          .thenAnswer((_) async => http.Response('{"title": "Test"}', 200));

      expect(await fetchPost(client), const TypeMatcher<Post>());
    });

    test('throws an exception if the http call completes with an error', () {
      final client = MockClient();

      // 제공된 http.Client를 호출했을 때, 실패 응답을 반환하기 위해 
      // Mockito를 사용합니다.
      when(client.get('https://jsonplaceholder.typicode.com/posts/1'))
          .thenAnswer((_) async => http.Response('Not Found', 404));

      expect(fetchPost(client), throwsException);
    });
  });
}
```

### 5. 테스트 수행하기

이제 테스트가 있는 `fetchPost()` 함수가 준비되었으니 실행해봅시다.

```terminal
$ dart test/fetch_post_test.dart
```

You can also run tests inside your favorite editor by following the
instructions in the [Introduction to unit testing][] recipe.

### 요약

In this example, you've learned how to use Mockito to test functions or classes
that depend on web services or databases. This is only a short introduction to
the Mockito library and the concept of mocking. For more information,
see the documentation provided by the [Mockito package][].


[Fetch data from the internet]: /docs/cookbook/networking/fetch-data
[Introduction to unit testing]: /docs/cookbook/testing/unit/introduction
[Mockito package]: {{site.pub-pkg}}/mockito
