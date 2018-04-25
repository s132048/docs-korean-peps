PEP: 3333
Title: Python Web Server Gateway Interface v1.0.1
Version: $Revision$
Last-Modified: $Date$
Author: P.J. Eby <pje@telecommunity.com>
Discussions-To: Python Web-SIG <web-sig@python.org>
Status: Final
Type: Informational
Content-Type: text/x-rst
Created: 26-Sep-2010
Post-History: 26-Sep-2010, 04-Oct-2010
Replaces: 333


PEP \333 독자를 위한 서문
=========================

이 문서는 PEP 333을 파이썬 3에서 더 잘사용할 수 있도록 약간 개선한 업데이트 버전이다.
WSGI 프로토콜에 대한 오래된 개정안들도 포함한다. (코드 샘플은 파이썬 3로 포팅되었다.)

절차상의 이유로 이 문서는 별개의 PEP가 되어야 한다.
파이썬 2 시리즈에서 호환되는 서버나 어플리케이션을 무효화하는 수정 사항은 없다.
파이썬 2 시리즈의 어플리케이션이나 서버가 PEP \333과 호환된다면 이 PEP와도 호환된다.

그러나 사용자의 어플리케이션이나 서버를 파이썬 3에서 사용하려면
이 섹션이 다루는 `문자열 타입에 대한 사항`_\ 과 `유니코드 문제`_\ 를 따라야 한다.

세부 정보가 필요하다면 이 문서와 PEP \333과의 라인 별 변경 사항을 리비전 84854의 SVN 리비전 기록을 본다.


개요
====

이 문서는 파이썬 웹 어플리케이션이나 프레임 워크와 웹 서버 간에 표준이 되는 인터페이스를 제안한다.
이를 통해 다양한 웹 서버에서 웹 어플리케이션의 이식 가능성을 향상한다.


기본이 되는 방법과 목표 (PEP \333로부터)
========================================

파이썬은 현재 Zope, Quixote, Webware, SkunkWeb, PSO, Twisted Web 등 다양한 웹 어플리케이션 프레임 워크를 지원한다.
([1]_ 참고.) 파이썬에 입문하는 사용자라면 오히려 이 다양성이 문제가 될 수 있다.
일반적으로 웹 프레임 워크의 선택으로 인해 사용 가능한 웹 서버에 제한이 생기기 때문이다.
반대로 웹 서버 선택으로 인해 웹 프레임 워크에 제한이 생길 수도 있다.

반면에 자바(Java)는 다수의 웹 어플리케이션 프레임 워크를 지원하지만 "servlet" API를 통해 어떤 자바
웹 어플리케이션 프레임 워크로 작성된 어플리케이션도 servlet API를 지원하는 모든 웹 서버에서 실행할 수 있다.

파이썬을 위한 웹서버 API의 가용성과 다양한 사용은 프레임 워크 선택과 웹 서버 선택을 분리함으로써
사용자가 적합한 프레임 워크와 웹 서버를 선택할 수 있게 한다. 이를 통해 프레임 워크 개발자와
웹 서버 개발자는 각자가 선호하는 전문 분야에 집중할 수 있다. 서버는 파이썬(Medusa 등)이나
임베디드 파이썬(mod_python 등)으로 작성되어도 되고 게이트웨이 프로토콜(CGI, FastCGI 등)으로 파이썬을 불러와도 된다.

따라서 이 PEP는 웹 서버와 웹 어플케이션이나 프레임 워크 간에 널리 사용되고 간단한 인터페이스인
WSGI(Python Web Server Gateway Interface)를 제안한다.

하지만 WSGI 사양만으로는 파이썬 웹 어플리케이션을 위한 서버와 프레임 워크의 기존 상태를 해결하지 못한다.
서버와 프레임워크 작성자와 관리자가 어떤 결과를 내려면 WSGI를 실제로 구현해야 한다.

기존 서버나 프레임 워크는 WSGI를 지원하지 않기 때문에 WSGI 지원을 구현한 작성자가 바로 얻을 수 있는 보상은 작다.
따라서 WSGI 구현은 반드시 쉬워야 하고 작성자가 초기에 인터페이스에 투자하는 비용이 합리적으로 낮아야 한다.

이러한 이유로 인터페이스의 서버와 프레임 워크 양면에서 구현이 간단해야 WSGI 인터페이스가 유용할 수 있고
간편함이 모든 설계 결정에 있어 중요한 기준이 된다.

그러나 프레임 워크 작성자의 입장에서 구현이 간편하다고 해서 웹 어플리케이션 작성자가 쉽게 사용할 수 있는 것은 아니다.
응답 객체와 쿠키 관리와 같은 것들이 기존 프레임 워크가 이를 해결하는 방식에 영향을 주기 때문에
WSGI는 꼭 필요한 인터페이스만을 프레임 워크 작성자에게 준다. 다시 말해, WSGI의 목적은 웹 프레임 워크를
새로 만들지 않고 기존 서버와 어플리케이션, 프레임 워크를 간편하게 연결하는 것이다.

이 목적으로 인해 WSGI는 배포된 파이썬 버전에서 사용할 수 없는 것을 요구하지 않는다. 새로운 표준 라이브러리
모듈은 이 사양에서 요구되거나 제안되지 않고 WSGI의 어떤 요소도 파이썬 2.2.2 버전 이상을 요구하지 않는다.
(그러나 이후의 파이선 버전을 위해 웹 서버에서 표준 라이브러리가 제공한 인터페이스의 지원을 포함하는 것이 좋다.)

기존 또는 향후의 프레임 워크와 서버를 위한 손쉬운 구현에 더해 서버에 어플리케이션이 될 수 있는 것이라면
요청 전처리기, 요청 후처리기, 그 외 WSGI 기반 "미들웨어" 컴포넌트를 쉽게 만들수 있어야 하고
동시에 어플리케이션을 위한 서버로 작동할 수 있어야 한다.

만약 미들웨어가 간단하고 견고해질 수 있고 WSGI가 서버와 프레임 워크에서 널리 사용될 수 있으면
완전히 새로운 종류의 파이썬 웹 어플리케이션 프레임 워크가 될 수 있다. 이는 느슨히 연결된
WSGI 미들웨어 컴포넌트로 구성된 프레임 워크를 말한다. 기존 프레임 워크 작성자는 자신의
프레임 워크의 기존 서비스가 이러한 방식으로 제공되도록 재구성할 수 있다. 이를 통해
WSGI와 사용되는 라이브러리와 같이 되고 일체형 프레임 워크를 벗어나게 된다.
이 방식은 어플리케이션 개발자가 특정 기능을 위해 잘구성된 컴포넌트를 선택하고
단일 프레임 워크의 단점을 극복할 수 있게 한다.

물론 이 문서를 작성하는 시점에서는 먼 미래의 일이지만 WSGI의 단기 목표인
어떤 서버와 어떤 프레임 워크도 사용할 수 있게 하는 것은 충분히 달성할 수 있다.

마지막으로 현재 WSGI 버전은 웹 서버와 서버 게이트웨이에 사용되는 어플리케이션 배포를 위한
메커니즘을 규정하지 않는다. 지금은 서버나 게이트웨이의 구현에 따라 정의된다.
충분히 많은 서버와 프레임 워크가 WSGI를 구현해 배포 사양에 대한 실무적인 경험이 쌓인 이후에
WSGI 서버와 어플리케이션 프레임 워크를 위한 배포 표준에 대한 PEP를 만들 수 있을 것이다.


사양 개요
=========

WSGI 인터페이스는 두가지 측면을 갖는다. 이는 "서버" 또는 "게이트웨이" 측면
그리고 "어플리케이션" 또는 "프레임 워크" 측면이다. 서버는 어플리케이션이 제공하는 호출 가능 객체를 불러온다.
이 객체가 어떻게 제공되는지는 서버나 게이트웨이에 따라 달라진다. 어떤 서버나 게이트웨이는 어플리케이션 배포자가
짧은 스크립트를 작성해 서버나 게이트웨이 인스턴스를 생성하도록 요구하고 이를 어플리케이션 객체와 함께 제공한다고 가정한다.
다른 서버와 게이트웨이는 설정 설정 파일이나 다른 메커니즘을 사용해 어플리케이션 객체를 받아오거나 얻는 위치를 지정할 수 있다.

순수한 서버/게이트웨이와 /어플리케이션/프레임 워크 외에도 이 두가지 사양을 구현하는 "미들웨어" 컴포넌트를 생성할 수 있다.
이 컴포넌트는 자신이 포함하는 서버에 어플리케이션처럼 작동하며 자신이 포함된 어플리케이션에는 서버처럼 작동한다.
또한 미들웨어 컴포넌트는 확장 API, 컨텐츠 변환, 탐색과 다른 유용한 기능을 제공할 수 있다.

이 사양에서 우리는 "호출 가능 객체"라는 용어를 "``__call__`` 메서드를 갖는 함수, 메서드, 클래스, 인스턴스"라는
의미로 사용한다. 호출 가능 객체를 구현하는 서버, 게이트웨이, 어플리케이션의 필요에 따라 알맞은 구현 기술을 선택한다.
반대로 호출 가능 객체를 호출하는 서버, 게이트웨이, 어플리케이션은 절대로 제공 받는 호출 객체의 종류에 의존성을 가져선 안된다.
호출 가능 객체는 호출만 될 수 있고 내부를 볼 수 없다.


문자열 타입에 대한 사항
-----------------------

일반적으로 HTTP는 바이트를 다룬다. 따라서 이 사양은 대부분 바이트를 다루는 방법에 대한 것이다.

그러나 이러한 바이트의 내용물은 어떤 텍스트 해석을 포함하는 경우가 많고
파이썬에서 문자열(string)은 텍스트를 다루는 가장 간편한 방법이다.

하지만 많은 파이썬 버전과 구현에서 문자열은 바이트가 아니라 유니코드다.
이로 인해 HTTP의 관점에서 텍스트와 바이트의 알맞은 변환과 사용 가능한 API 간에 세심한 균형이 요구된다.
특히 다양한 ``str`` 타입을 포함하는 파이썬 구현 간에 코드 포팅을 지원할 때 그렇다.

따라서 WSGI 두가지 종류의 "문자열"을 정의한다.

* 항상 ``str`` 타입으로 구현된 "네이티브" 문자열은 요청/응답 헤더와 메타 데이터에 사용된다.

* 파이썬 3에서는 ``bytes`` 타입, 다른 곳에서는 ``str``\ 으로 구현된 "바이트 문자열(bytestring)"은
  요청의 바디와 응담에 사용된다. (예시: POST/PUT 입력 데이터, HTML 페이지 출력.)

그러나 혼동해선 안되는 것이 있다. 파이썬의 ``str`` 타입이 실제로는 유니코드일지라도
네이티브 문자열의 내용은 반드시 Latin-1 인코딩으로 바이트로 변환될 수 있어야 한다.
(이 문서 뒷부분의 `유니코드 문제`_\ 에서 자세한 내용을 볼 수 있다.)

간단히 말해 이 문서에서 "문자열"이란 객체의 타입이 ``str``\ 인 네이티브 문자열을 말한다.
내부적으로 유니코드로 구현되었는지 바이트로 구현되었는지는 상관없다.
바이트 문자열이라는 용어는 파이썬 3에서는 ``bytes``, 파이썬 2에서는 ``str`` 타입인 객체를 말한다.

HTTP가 어떤 의미에서 실제로 바이트일지라도 파이썬의 기본 ``str`` 타입에 관계없이 사용할 수 있는 많은 API가 있다.



어플리케이션/프레임워크 관점
----------------------------

어플리케이션 객체(object)는 간단히 말하면 두개의 인수를 받는 호출 가능한 객체다.
"객체"라는 용어를 실제 객체 인스턴스를 요구하는 것으로 오해해선 안된다.
``__call__`` 메서드를 갖는 모든 함수, 메서드, 클래스, 인스턴스를 어플리케이션 객체로 사용할 수 있다.
CGI 외에 거의 모든 서버/게이트웨이가 반복적으로 리퀘스트를 하므로
어플리케이션 객체는 반드시 한번 이상 호출될 수 있어야 한다.

(주의: "어플리케이션" 객체라고 부르고 있지만 어플리케이션 개발자가 WSGI를 웹 프로그래밍 API로 사용한다는 의미는 아니다.
어플리케이션 개발자가 기존의 상위 레벨 프레임 워크 서비스로 어플리케이션을 개발할 수 있다는 것을 가정한다.
WSGI는 프레임 워크와 서버 개발자를 위한 도구로 어플리케이션 개발자를 직접 지원하기 위해 고안된 것은 아니다.)

다음은 어플리케이션 객체에 대한 두가지 예시로 하나는 함수 나머지 하나는 클래스다. ::

    HELLO_WORLD = b"Hello world!\n"

    def simple_app(environ, start_response):
        """Simplest possible application object"""
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        start_response(status, response_headers)
        return [HELLO_WORLD]

    class AppClass:
        """Produce the same output, but using a class

        (Note: 'AppClass' is the "application" here, so calling it
        returns an instance of 'AppClass', which is then the iterable
        return value of the "application callable" as required by
        the spec.

        If we wanted to use *instances* of 'AppClass' as application
        objects instead, we would have to implement a '__call__'
        method, which would be invoked to execute the application,
        and we would need to create an instance for use by the
        server or gateway.
        """

        def __init__(self, environ, start_response):
            self.environ = environ
            self.start = start_response

        def __iter__(self):
            status = '200 OK'
            response_headers = [('Content-type', 'text/plain')]
            self.start(status, response_headers)
            yield HELLO_WORLD


서버/게이트웨이 측면
--------------------

서버나 게이트웨이는 HTTP 클라이언트로부터 요청을 받을 때마다 어플리케이션 호출 가능 객체를 불러온다.
이 과정은 어플리케이션에서 유도된다. 다음은 설명을 위한 간단한 CGI 게이트웨이 예시로 어플리케이션
객체를 받는 함수로 구현되어 있다. 이 예시에서 확인되지 않은 예외는 기본으로 ``sys.stderr``\ 에
덮어쓰여지고 웹 서버에 기록되므로 제한적인 에러 처리만을 할 수 있다.

::

    import os, sys

    enc, esc = sys.getfilesystemencoding(), 'surrogateescape'

    def unicode_to_wsgi(u):
        # Convert an environment variable to a WSGI "bytes-as-unicode" string
        return u.encode(enc, esc).decode('iso-8859-1')

    def wsgi_to_bytes(s):
        return s.encode('iso-8859-1')

    def run_with_cgi(application):
        environ = {k: unicode_to_wsgi(v) for k,v in os.environ.items()}
        environ['wsgi.input']        = sys.stdin.buffer
        environ['wsgi.errors']       = sys.stderr
        environ['wsgi.version']      = (1, 0)
        environ['wsgi.multithread']  = False
        environ['wsgi.multiprocess'] = True
        environ['wsgi.run_once']     = True

        if environ.get('HTTPS', 'off') in ('on', '1'):
            environ['wsgi.url_scheme'] = 'https'
        else:
            environ['wsgi.url_scheme'] = 'http'

        headers_set = []
        headers_sent = []

        def write(data):
            out = sys.stdout.buffer

            if not headers_set:
                 raise AssertionError("write() before start_response()")

            elif not headers_sent:
                 # Before the first output, send the stored headers
                 status, response_headers = headers_sent[:] = headers_set
                 out.write(wsgi_to_bytes('Status: %s\r\n' % status))
                 for header in response_headers:
                     out.write(wsgi_to_bytes('%s: %s\r\n' % header))
                 out.write(wsgi_to_bytes('\r\n'))

            out.write(data)
            out.flush()

        def start_response(status, response_headers, exc_info=None):
            if exc_info:
                try:
                    if headers_sent:
                        # Re-raise original exception if headers sent
                        raise exc_info[1].with_traceback(exc_info[2])
                finally:
                    exc_info = None     # avoid dangling circular ref
            elif headers_set:
                raise AssertionError("Headers already set!")

            headers_set[:] = [status, response_headers]

            # Note: error checking on the headers should happen here,
            # *after* the headers are set.  That way, if an error
            # occurs, start_response can only be re-called with
            # exc_info set.

            return write

        result = application(environ, start_response)
        try:
            for data in result:
                if data:    # don't send headers until body appears
                    write(data)
            if not headers_sent:
                write('')   # send headers now if body was empty
        finally:
            if hasattr(result, 'close'):
                result.close()


Middleware: 양쪽에 작용하는 요소
--------------------------------

단일 객체는 어떤 어플리케이션에 대한 서버의 역할을 수행하면서 어떤 서버에 대한 어플리케이션이 될 수 있다.
"미들웨어" 컴포넌트가 이러한 기능을 수행한다.

* ``environ``\ 을 재작성한 후에 대상 URL에 기반해 여러 어플리케이션 객체로의 요청을 라우팅한다.

* 다수의 어플리케이션이나 프레임 워크가 같은 프로세스에서 나란히 실행될 수 있게 한다.

* 네트워크를 통한 요청과 응답 전달을 통 밸런싱과 원격 처리를 로드 밸런싱과 원격 처리.

* XSL 스타일시트를 적용하는 것과 같이 컨텐츠 후처리를 수행한다.

일반적으로 미들웨어의 존재는 "서버/게이트웨이"와 "어플리케이션/프레임워크" 양쪽 인터페이스에
투명하므로 특별한 지원을 필요로 하지 않는다. 사용자가 미들웨어를 어플리케이션에 통합하고자 한다면
미들웨어 컴포넌트를 서버에 어플리케이션처럼 제공한다. 그리고 미들웨어 컴포넌트가
서버처럼 어플리케이션을 불러오도록 설정한다. 물론 미들웨어가 래핑하는 "어플리케이션"은 실제로
다른 어플리케이션을 래핑하는 미들웨어 컴포넌트가 될 수도 있다. 이를 통해 "미들웨어 스택"이라는 것을 생성한다.

대부분의 미들웨어는 반드시 WSGI의 서버와 어플리케이션 측면이 갖는 제한과 요구 사항을 따라야 한다.
그러나 종종 미들웨어에 대한 요구 사항은 순수한 서버나 어플리케이션의 요구 사항보다 엄격할 수 있고
이러한 내용은 사양에 명시되어 있을 것이다.

다음 예시는 조 스트라우트의 ``piglatin.py`` 코드를 사용해 ``text/plain`` 응답을
피그 라틴으로 변환하는 미들웨어 컴포넌트다. (주의: 실제 미들웨어 컴포넌트는
컨텐츠 타입을 검사할 때 더 강력한 방법을 사용하고 컨텐츠의 인코딩도 검사해야 한다.
또한 이 예시는 단어가 블럭 경계를 넘어 쪼개지는 경우를 무시한다.)

::

    from piglatin import piglatin

    class LatinIter:

        """Transform iterated output to piglatin, if it's okay to do so

        Note that the "okayness" can change until the application yields
        its first non-empty bytestring, so 'transform_ok' has to be a mutable
        truth value.
        """

        def __init__(self, result, transform_ok):
            if hasattr(result, 'close'):
                self.close = result.close
            self._next = iter(result).__next__
            self.transform_ok = transform_ok

        def __iter__(self):
            return self

        def __next__(self):
            if self.transform_ok:
                return piglatin(self._next())   # call must be byte-safe on Py3
            else:
                return self._next()

    class Latinator:

        # by default, don't transform output
        transform = False

        def __init__(self, application):
            self.application = application

        def __call__(self, environ, start_response):

            transform_ok = []

            def start_latin(status, response_headers, exc_info=None):

                # Reset ok flag, in case this is a repeat call
                del transform_ok[:]

                for name, value in response_headers:
                    if name.lower() == 'content-type' and value == 'text/plain':
                        transform_ok.append(True)
                        # Strip content-length if present, else it'll be wrong
                        response_headers = [(name, value)
                            for name, value in response_headers
                                if name.lower() != 'content-length'
                        ]
                        break

                write = start_response(status, response_headers, exc_info)

                if transform_ok:
                    def write_latin(data):
                        write(piglatin(data))   # call must be byte-safe on Py3
                    return write_latin
                else:
                    return write

            return LatinIter(self.application(environ, start_latin), transform_ok)


    # Run foo_app under a Latinator's control, using the example CGI gateway
    from foo_app import foo_app
    run_with_cgi(Latinator(foo_app))



사양 세부 사항
==============

어플리케이션 객체는 반드시 두개의 위치 인수를 받는다. 설명을 위해 이 인수를
``environ``\ 과 ``start_response``\ 로 이름 붙였지만 다른 이름을 사용해도 된다.
서버나 게이트웨이는 반드시 키워드가 아닌 위치 인수를 사용해 어플리케이션 객체를 불러와야 한다.
(예를 들어, 위 예시의 ``result = application(environ, start_response)``\ 와 같이)

``environ`` 매개 변수는 딕셔너리 객체로 CGI 스타일 환경 변수를 포함한다.
이 객체는 반드시 파이썬 기본 제공 딕셔너리가 되어야 한다. (서브클래스 ``UserDict``\ 나
다른 딕셔너리 형태여서는 안된다.) 그리고 어플리케이션은 원하는 방식으로 이 딕셔너리를 수정할 수 있다.
추가로 딕셔너리는 WSGI가 요구하는 특정한 변수를 포함해야 한다. 이 변수들에 대해서는 이후에 다룬다.
딕셔너리는 아래에서 다룰 규칙에 따라 명명된 서버 특정 확장 변수를 포함할 수 있다.

``start_response`` 매개 변수는 두개의 필수 위치 인수와 선택 인수를 받는 호출 가능 객체다.
설명을 위해 이 인수를 ``status``, ``response_headers``, ``exc_info``\ 로 이름 붙였지만
다른 이름을 사용해도 된다. 어플리케이션은 반드시 위치 인수를 사용해 ``start_response`` 객체를 불러와야 한다.
(예를 들어, 위 예시의 ``start_response(status, response_headers)``\ 와 같이)

``status`` 매개 변수는 ``"999 Message here"`` 형태의 상태 문자열이고
``response_headers``\ 은 HTTP 응답 헤더를 나타내는 ``(header_name, header_value)``
튜플로 이루어진 리스트다. 선택 매개 변수 ``exc_info``\ 는 아래의
```start_response()`` 호출 가능 객체`_\ 와 `에러 처리`_ 섹션에서 다룬다.
이 매개 변수는 어플리케이션에 에러가 나타나고 브라우저에 에러 메세지를 표시하려 할 때만 사용된다.

``start_response`` 호출 가능 객체는 반드시 ``write(body_data)`` 호출 가능 객체를 반환해야 한다.
이 객체는 HTTP 응답 바디의 일부로 적성된 바이트 문자열을 위치 매개 변수로 받는다.
(주의: ``write()`` 호출 가능 객체는 특정한 기존 프레임 워크의 명령형 출력 API를 지원하지 위해서만 제공된다.
가능하다면 새로운 어플리케이션이나 프레임 워크에 의해 사용되면 안된다. `버퍼링과 스트리밍`_ 섹션에서
관련된 세부 사항을 볼 수 있다.)

서버에 의해 호출될 때 어플리케이션 객체는 0 이상의 바이트 문자열을 반환하는 반복 가능 객체를 반환해야 한다.
바이트 문자열 리스트를 반환하거나 바이트 문자열을 반환하는 생성자 역할을 하는 어플리케이션 또는
반복 가능한 인스턴스를 갖는 클래스를 어플리케이션으로 사용하는 등 다양한 방법으로 이를 구현할 수 있다.
방법에 관계 없이 어플리케이션 객체는 반드시 0 이상의 바이트 문자열을 반환하는 반복 가능 객체를 반환해야 한다.

서버나 게이트웨이는 반드시 반환된 바이트 문자열을 버퍼되지 않게 클라이언트로 보내
다른 요청이 들어오기 전에 바이트 문자열 전송을 마쳐야 한다.
(다시 말해 어플리케이션은 자신의 버퍼링을 수행해야 한다. `버퍼링과 스트리밍`_ 섹션에서
어플리케이션 출력을 관리하는 방법에 대한 추가 정보를 볼 수 있다.)

서버나 게이트웨이는 반환된 바이너리 문자열을 바이트 시퀀스로 취급해야 한다.
특히 라인의 끝이 변경되지 않는 다는 것을 보장해야 한다.
어플리케이션은 바이트 문자열이 형식이 클라이언트에 맞는 형식이 되도록 보장해야 한다.
(서버나 게이트웨이는 HTTP 전송 인코딩을 적용하거나 바이트 레인지 전송과 같이
HTTP 기능 구현을 목적으로 다른 전송을 수행할 수 있다. `기타 HTTP 기능`_\ 에서 자세한 정보를 볼 수 있다.)

``len(iterable)``\ 로의 호출을 성공하면 서버는 결과의 정확성에 의존할 수 있어야 한다.
이는 어플리케이션이 반환한 반복 가능 객체가 작동하는 ``__len__()`` 메서드를 제공했을 때
정확한 결과를 반환한다는 것이다. (```Content-Length`` 헤더 관리`_ 섹션에서 이 메서드가 어떻게
사용되는지에 대한 정보를 볼 수 있다.)

만약 어플리케이션이 반환한 반복 가능 객체가 ``close()`` 메서드를 가지면
요청이 정상적으로 종료되었는지 반복 중 어플리케이션 에러 또는 브라우저의 연결 해제로
인해 일찍 중단되었는지에 관계 없이 서버나 게이트웨이는 반드시 현재 요청을 완료하고
이 메서드를 호출해야 한다. (``close()`` 메서드의 요구 사항은 어플리케이션에 의한 리소스
해방을 지원하는 것이다. 이 프로토콜은 PEP 342의 생성자 지원과 ``close()`` 메서더를 갖는
다른 일반 반복 가능 객체를 보완하기 위함이다.)

생성자를 반환하는 어플리케이션이나 다른 커스텀 반복자는 반복자가 모두 사용될 것으로 가정해선 안된다.
또한 서버에 의해 일찍 중단될 수 있다.

(주의: 반복 가능 객체의 첫번째 바디 바이트 문자열을 반환하기 전에 어플리케이션은 반드시
``start_response()`` 호출 가능 객체를 호출해야 한다. 서버가 어떤 바디 내용보다 먼저
헤더를 보내기 위함이다. 그러나 이 호출은 반복 가능 객체의 첫번째 반복에 의해 수행될 수 있다.
따라서 서버가 반복 가능 객체의 반복을 시작하기 전에 ``start_response()``\ 가 호출된다고 가정해선 안된다.)

마지막으로 서버와 게이트웨이가 어플리케이션이 반환한 반복 가능 객체의 다른 속성을 직접 사용해선 안된다.
이는 ``wsgi.file_wrapper``\ 가 반환하는 "file wrapper"와 같이 반복 가능 객체가 서버나 게이트웨이 타입에
특정한 인스턴스인 경우에만 가능하다. 일반적인 경우에 이 문서에서 지정한 속성이나 PEP 234 반복 API를 통해
접근한 속성만 가능하다.


``environ`` 변수
----------------

``environ`` 딕셔너리를 다음 CGI 변수들을 포함해야 한다.
변수들은 CGI(Common Gateway Interface) 사양\ [2]_\ 에 의해 정의된다.
다음 변수들은 그 값이 빈 문자열이 아닌 한 반드시 있어야 한다.
값이 빈 문자열이면 변수에 따라 생략될 수 있다.

``REQUEST_METHOD``
  HTTP 요청 방법. ``"GET"``\ 이나 ``"POST"``. 빈 문자열이 올 수 없는 필수 변수다.

``SCRIPT_NAME``
  요청 URL "경로"의 앞부분으로 어플리케이션 객체에 대응된다.
  이 변수로 어플리케이션이 자신의 가상 "위치"를 알게 된다.
  만약 어플리케이션이 서버의 "루트"에 대응된다면 빈 문자열이 올 수 있다.

``PATH_INFO``
  요청 URL "경로"의 나머지 부분. 어플리케이션 내부에 있는 요청 대상의 가상 "위치"를 가리킨다.
  요청 URL 대상이 어플리케이션 루트고 이후에 슬래시가 오지 않으면 빈 문자열이 올 수 있다.

``QUERY_STRING``
  요청 URL에서 ``"?"`` 이후에 오는 부분. 빈 문자열이 오거나 생략될 수 있다.

``CONTENT_TYPE``
  HTTP 리퀘스트의 ``Content-Type`` 필드에 들어가는 모든 내용.
  빈 문자열이 오거나 생략될 수 있다.

``CONTENT_LENGTH``
  HTTP 리퀘스트의 ``Content-Length`` 필드에 들어가는 모든 내용.
  빈 문자열이 오거나 생략될 수 있다.

``SERVER_NAME``, ``SERVER_PORT``
  ``SCRIPT_NAME``, ``PATH_INFO``\ 와 합쳐지면 이 두 문자열은 전체 URL로 사용된다.
  그러나 ``HTTP_HOST``\ 가 있으면 요청 URL을 재작성할 때 ``SERVER_NAME``\ 에 우선하여 사용되어야 한다.
  `URL 재작성`_ 섹션에서 자세한 정보를 본다. ``SERVER_NAME``\ 과 ``SERVER_PORT``\ 는
  빈 문자열이 되선 안되는 필수 변수다.

``SERVER_PROTOCOL``
  클라이언트가 요청을 보낼 때 사용한 프로토콜 버전.
  일반적으로 ``"HTTP/1.0"``\ 이나 ``"HTTP/1.1"`` 같은 것이 온다.
  어플리케이션은 이 변수를 사용해 HTTP 요청을 처리하는 방법을 정할 수 있다.
  (이 변수는 요청에 사용된 프로토콜을 알려주므로 ``REQUEST_PROTOCOL``\ 로
  불려야 하고 서버의 응답에 사용되는 프로토콜일 필요는 없다. 그러나 CGI와의
  호환성을 위해 기존 명칭을 따른다.)

``HTTP_`` Variables
  클라이언트 제공 HTTP 요청 헤더에 대응되는 변수. (예시: 이름이 ``"HTTP_"``\ 로 시작하는 변수.)
  이 변수의 유무는 요청의 적절한 HTTP 헤더 유무와 일치해야 한다.

서버나 게이트웨이는 적용 가능한 CGI 변수를 최대한 제공해야 한다.
추가로 만약 SSL이 사용되면 서버나 게이트웨이는 ``HTTPS=on``\ 와
``SSL_PROTOCOL`` 같이 적용 가능한 아파치 SSL 환경 변수를 [5]_
최대한 많이 제공해야 한다. 그러나 위에 나열된 것 대신 CGI 변수를 사용하는
어플리케이션은 관련 확장을 지원하지 않는 웹 서버로 이식할 수 없다.
(예를 들어, 파일을 게시하지 않는 제대로 된 ``DOCUMENT_ROOT``\ 나
``PATH_TRANSLATED``\ 를 제공할 수 없을 것이다.)

WSGI 지원 서버나 게이트웨이는 제공하는 변수와 그 정의를 문서화 해야 한다.
어플리케이션은 요구하는 변수의 유무를 확인하고 변수가 없는 경우를 위한 예비책을 가져야 한다.

주의: 인증이 발생하지 않을 때의 ``REMOTE_USER`` 변수와 같이 누락 변수는
``environ`` 딕셔너리에서 빠져야 한다. 또한 CGI 정의 변수가 있을 때는
네이티브 문자열이 되어야 한다. 어떤 CGI 변수값이 ``str``\ 이 아닌 타입을 가지면 사양 위반이다.

CGI 정의 변수에 더해 ``environ`` 딕셔너리는 임의의 운영 체제 "환경 변수"를 포함할 수 있다.
또한 다음 WSGI 정의 변수를 반드시 포함해야 한다.

=====================  ===============================================
Variable               Value
=====================  ===============================================
``wsgi.version``       WSGI 버전 1.0을 나타내는 튜플 ``(1, 0)``.

``wsgi.url_scheme``    어플리케이션이 호출된 URL의 "스키마" 부분을
                       나타내는 문자열. 일반적으로 ``"http"`` 또는
                       ``"https"`` 중 적절한 값이 된다.

``wsgi.input``         HTTP 요청 바디 바이트를 읽어올 수 있는
                       입력 스트림으로 객체와 같이 보인다.
                       (서버나 게이트웨이는 요청에 따라 읽기를
                       수행하거나 클라이언트의 요청 바디를 먼저
                       읽고 메모리나 디스크에 버퍼할 수 있고
                       입력 스트림을 제공하기 위한 다른 기술을 사용할
                       수 있다. 선호도에 따라 결정한다.)

``wsgi.errors``        에러 출력이 작성되는 출력 스트림으로 객체와
                       같이 보인다. 프로그램이나 다른 에러를
                       표준화되고 가능한 중앙화된 위치에 기록하기
                       위한 목적으로 사용된다. "텍스트 모드"
                       스트림이어야 한다. 예를 들어, 어플리케이션은
                       행 끝에 ``"\n"``\ 를 사용하고 서버/게이트웨이에
                       의해 적절한 행 끝으로 변환된다고 가정한다.

                       (``str`` 타입이 유니코드인 플랫폼에서
                       에러 스트림은 임이의 유니코드를 에러 없이
                       받고 로그해야 한다. 스트림의 인코딩에서
                       렌러딩할 수 없는 문자는 대체할 수 있다.)

                       많은 서버에서 ``wsgi.errors``\ 가 서버의
                       메인 에러 로그가 된다. 대안으로 ``sys.stderr``
                       또는 어떤 종류의 로그 파일이 될 수 있다.
                       서버의 문서는 이에 대한 설정 방법이나 기록된
                       출력을 찾을 수 있는 위치에 대한 정보를
                       포함해야 한다. 서버나 게이트웨이는 원할 경우
                       다른 에러 스트림을 다른 어플리케이션에 제공할 수 있다.

``wsgi.multithread``   어플리케이션 객체가 같은 프로세스의
                       다른 스레드에 의해 동시에 호출되려면
                       True가 되어야 한다. 아닌 경우엔 False다.

``wsgi.multiprocess``  어플리케이션 객체가 다른 프로세스에 의해
                       동시에 호출될 수 있으면 True가 되어야 한다.
                       아닌 경우엔 False다.

``wsgi.run_once``      서버나 게이트웨이가 어플리케이션이 포함된
                       프로세스의 수명 주기 동안 True가 되어야 한다.
                       일반적으로 이 값은 CGI 기반 게이트웨이나
                       이와 유사한 것일 때 True가 된다.
=====================  ===============================================

마지막으로 ``environ`` 딕셔너리는 서버 정의 변수를 포함할 수 있다.
이 변수들은 소문자, 숫자, 점, 언더스코어만을 사용한 이름이어야 하고
정의하는 서버나 게이트웨이의 고유한 이름을 접두어로 사용해야 한다.
예를 들어, ``mod_python``\ 이 정의한 변수는 ``mod_python.some_variable``\ 과
같은 이름이 된다.


입력과 에러 스트림
~~~~~~~~~~~~~~~~~~

서버가 제공하는 입력과 에러 스트림은 다음 메서드를 지원해야 한다.

===================  ==========  ========
Method               Stream      Notes
===================  ==========  ========
``read(size)``       ``input``   1
``readline()``       ``input``   1, 2
``readlines(hint)``  ``input``   1, 3
``__iter__()``       ``input``
``flush()``          ``errors``  4
``write(str)``       ``errors``
``writelines(seq)``  ``errors``
===================  ==========  ========

몇몇 예외를 제외하고 각 메서드의 의미는 파이썬 라이브러리 레퍼런스에 문서화된 것과 같다.
예외는 위 테이블에 노트 번호에 해당하며 다음과 같다.

1. 서버는 이전 클라이언트 지정 ``Content-Length``\ 를 읽지 않아도 된다.
   또한 어플리케이션이 이전 포인트를 읽으려 한다면 동시에 파일 끝 조건을 시뮬레이트 해야 한다.
   어플리케이션은 ``CONTENT_LENGTH`` 변수에 지정된 것보다 많은 데이터를 읽으려 해선 안된다.

   서버는 인수없이 ``read()``\ 가 호출되는 것을 허용하고 클라이언트 입력 스트림의 나머지 부분을 반환해야 한다.

   서버는 비어 있거나 소모된 입력 스트림을 읽으려는 모든 시도에 대해 빈 바이트 문자열을 반환해야 한다.

2. 서버는 ``readline()``\ 에 "size" 선택 인수를 지원해야 하지만 WSGI 1.0에서는 지원을 생략해도 된다.

   (WSGI 1.0에서 사이즈 인수는 지원되지 않는다. 구현이 복잡하고 실제로 자주 사용되지 않기 때문이다.
   하지만 ``cgi`` 모듈이 사용을 시작했기 때문에 실제 서버는 지원해야 한다.)

3. 호출자와 구현자에서 ``readlines()``\ 의  ``hint``\ 는 선택 인수다.
   어플리케이션은 이를 제공하지 않아도 되고 서버나 게이트웨이는 이를 무시해도 된다.

4. ``errors`` 스트림을 되돌릴 수 없기 때문에 서버와 게이트웨이는 작성 처리를 버퍼링 없이 바로 보내도 된다.
   이 경우에 ``flush()`` 메서드는 무동작이 될 수 있다. 그러나 포터블 어플리케이션의 경우 출력이 버퍼되지 않고
   ``flush()`` 메서드가 무동작이 된다고 가정할 수 없다. 출력이 실제로 작성된다는 것을 보장해야 한다면
   포터블 어플리케이션은 반드시 ``flush()`` 메서드를 호출해야 한다. (예를 들어, 같은 에러 로그를 작성하는 여러
   프로세스로부터 온 데이터가 혼재되는 것을 최소화 하길 원하는 경우가 이에 해당한다.)

이 사양을 따르는 모든 서버는 위 테이블에 나열된 메서드를 지원해야 하고
``input``\ 이나 ``errors``\ 의 다른 메서드와 속성을 사용해선 안된다.
특히 어플리케이션은 이 스트림이 ``close()`` 메서드를 가지고 있더라도 닫으려 해선 안된다.


``start_response()`` 호출 가능 객체
-----------------------------------

어플리케이션 객체로 보내지는 두번째 매개 변수는 ``start_response(status, response_headers, exc_info=None)``
형태의 호출 가능 객체다. (다른 WSGI 호출 가능 객체처럼 인수는 키워드 인수가 아닌
위치 인수로 주어져야 한다.) ``start_response`` 호출 가능 객체는 HTTP 응답을 시작하기 위해
사용되고 반드시 ``write(body_data)`` 호출 가능 객체를 반환해야 한다.
(`버퍼링과 스트리밍`_ 섹션을 참고한다.)

``status``인수는 ``"200 OK"``\ 나  ``"404 Not Found"`` 같은 HTTP "상태" 문자열이다.
이는 Status-Code와 Reason-Phrase의 순서로 구성되고 하나의 공백으로 구분된다.
양쪽에 공백이나 다른 문자가 오지 않는다. (RFC 2616의 6.1.1 섹션에 자세한 정보가 있다.)
문자열은 제어 문자를 포함해선 안되고 캐리지 리턴, 라인피드나 이 조합으로 끝나선 안된다.

``response_headers`` 인수는 ``(header_name, header_value)`` 튜플 리스트다.
서버는 이 인수의 내용을 원하는 방식으로 바꿀 수 있다. 각 ``header_name``\ 은
반드시 유효한 HTTP 헤더 필드명이 되어야 하고 콜론이나 구두점이 와선 안된다.
(RFD 2616 섹션 4.2에 정의되어 있다.)

각 ``header_value``\ 는 캐리지 리턴이나 라인피드를 포함한 어떤 제어 문자도 포함해선 안되고
이러한 것들이 중간이나 끝에 나와서도 안된다. (이 요구 사항은 요청 헤더를 검사하거나 수정해야
하는 서버, 게이트웨이, 중간 응답 프로세서에 의해 수행되는 모든 파싱의 복잡성을 최소화하기 위함이다.)

일반적으로 서버나 게이트웨이는 클라이언트에 알맞는 헤더가 보내진다는 것을 보장해야 한다.
만약 어플리케이션이 HTTP나 관련된 다른 사양이 요구하는 헤더를 생략하면 서버나 게이트웨이가
이를 추가해야 한다. 예를 들어, HTTP ``Date:``\ 와 ``Server:`` 헤더는
일반적으로 서버나 게이트웨이가 추가한다.

(서버/게이트웨이 작성자를 위한 알림: HTTP 헤더명은 대소문자를 구분하지 않기 때문에
어플리케이션 제공 헤더를 검사할 때 이를 고려해야 한다.)

어플리케이션과 미들웨어에서  HTTP/1.1 "홉 바이 홉" 기능이나 헤더, HTTP/1.0의
동등한 기능, 클라이언트의 웹 서버 연결의 지속성에 영향을 줄수 있는 헤더의 사용은 금지되어 있다.
이 기능들은 실제 웹 서버만의 영역이다. 이 기능들은 이를 보내려는 어플리케이션에서
심각한 에러가 된다. 서버나 게이트웨이는 이 기능들이 ``start_response()``\ 에 제공됐을 때
에러를 발생시켜야 한다. (홉 바이 홉 기능과 헤더에 대한 자세한 정보는 `기타 HTTP 기능`_\ 을 본다.)

``start_response``\ 가 호출된 시점에 서버는 헤더의 에러를 확인해 어플리케이션이
실행되는 도중에 에러를 발생시킬 수 있게 해야 한다.

그러나 ``start_response`` 호출 가능 객체는 응답 헤더를 실제로 보내서는 안된다.
대신 응답 헤더를 저장해 비어있지 않은 바이트 문자열을 반환하는
어플리케이션 반환 값의 첫번째 반복이나 어플리케션이 ``write()`` 호출 가능 객체를
처음 호출한 이후에만 서버나 게이트웨이가 응답 헤더를 보낼 수 있게 해야 한다.
다시 말해 실제 바디 데이터가 있거나 어플리케이션의 반환된 반복 가능 객체가
소모된 후에만 응답 헤더가 보내져야 한다. (응답 헤더의 ``Content-Length``\ 가
명시적으로 0인 경우에만 예외로 응답 헤더를 보낼 수 있다.)

이러한 응답 헤더 전송의 지연은 버퍼링되거나 비동기화인 어플리케이션이 처음에 의도된
출력을 가능한 마지막 순간까지 에러 출력으로 대체할 수 있다는 것을 보장하기 위함이다.
예를 들어, 어플리케이션 버퍼에서 바디가 생성되는 도중에 에러가 발생하면
응답 상태를 "200 OK"에서 "500 Internal Error"로 바꿔야 한다.

``exc_info`` 인수가 제공되면 이 인수는 반드시 파이썬 ``sys.exc_info()`` 튜플이어야 한다.
이 인수는 에러 핸들러에 의해 ``start_response``\ 가 호출될 때만 어플리케이션에 의해 제공된다.
``exc_info``\ 가 제공되고 출력된 HTTP 헤더가 없다면 ``start_response``\ 는 현재 저장된
HTTP 응답 헤더를 새로 제공된 것으로 대체해야 한다. 이를 통해 어플리케이션은 에러가 발생했을 때
출력을 바꿀 수 있다.

그러나 HTTP 헤더가 이미 보내지고 ``exc_info``\ 가 제공되면 ``start_response``\ 는
에러를 발생시키고 ``exc_info`` 튜플을 사용해 다시 에러를 발생시켜야 한다. ::

    raise exc_info[1].with_traceback(exc_info[2])

위 코드는 어플리케이션이 포착한 예외를 다시 발생시키고 원칙적으로 어플리케이션을 중단시켜야 한다.
(HTTP 헤더가 이미 보내진 이후에 어플리케이션이 브라우저에 에러 출력을 하려는 시도는 안전하지 않다.)
어플리케이션이 ``exc_info``\ 와 ``start_response``\ 를 호출하면 어플리케이션은
``start_response``\ 에 의해 발생된 에러를 포착해선 안된다. 대신 이러한 예외가 서버나 게이트웨이로
전달될 수 있게 해야 한다. `에러 처리`_ 섹션에서 자세한 정보를 볼 수 있다.

``exc_info`` 인수가 제공됐을 때만 어플리케이션은 ``start_response``\ 를 한번 이상 호출할 수 있다.
정확히 말하면 ``start_response``\ 가 어플리케이션의 현재 호출에서 불러진 이후에
``exc_info`` 없이 ``start_response``\ 를 호출하는 것은 심각한 에러다.
이는 ``start_response``\ 로의 첫번째 호출이 에러를 발생시킨 경우를 포함한다.
(정확한 로지겡 대한 설명은 위의 CGI 게이트웨이 예시를 본다.)

주의: ``start_response``\ 를 구현하는 서버, 게이트웨이, 미들웨어는 함수의 실행 기간 동안
``exc_info`` 매개 변수로의 참조가 없다는 것을 보장해야 한다. 트레이드백과 관련된 프레임을 통한
순환 참조가 생성되는 것을 막기 위함이다. 이를 구현하는 가장 간단한 방법은 다음과 같다. ::

    def start_response(status, response_headers, exc_info=None):
        if exc_info:
             try:
                 # do stuff w/exc_info here
             finally:
                 exc_info = None    # Avoid circular ref.

CGI 게이트웨이 예시는 이러한 기법에 대한 다른 설명을 제공한다.


``Content-Length`` 헤더 관리
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

어플리케이션이 ``Content-Length`` 헤더를 제공하면 헤더가 허용하는 것보다 많은 바이트를
클라이언트로 보내면 안된다. 추가로 충분한 데이터가 보내졌을 때 요청에 대한 반복을 중단해야 한다.
그리고 어플리케이션이 그 시점 이전에 ``write()``\ 를 시도하면 에러를 발생시켜야 한다.
(물론 어플리케이션이 선언된 ``Content-Length``\ 를 충족하지 못하는 데이터를 보내면
서버는 연결을 중단하고 로그하거나 에러를 보고해야 한다.)

어플리케이션이 ``Content-Length`` 헤더를 제공하지 않으면 서버나 게이트웨이는
이를 처리하기 위한 여러 방법 중 하나를 고를 수 있다. 가장 간단한 방법은
요청이 완료되었을 때 클라이언트 연결을 닫는 것이다.

그러나 상황에 따라 서버나 게이트웨이가 ``Content-Length`` 헤더를 생성하거나
최소한 클라이언트 연결을 닫아야 하는 상황을 피할 수 있다.
어플리케이션이 ``write()`` 호출 가능 객체를 호출하지 않고 ``len()``\ 가 1이 되는
반복 가능 객체를 반환하면 이 객체가 반환한 첫번째 바이트 문자열의 길이를 사용함으로써
서버는 자동으로 ``Content-Length``\ 를 결정할 수 있다.

그리고 서버와 클라이언트가 모두 HTTP/1.1 "청크 인코딩"을 지원하면 서버는 청크 인코딩을 사용해
각 ``write()`` 호출이나 반복 가능 객체가 반환한 바이트 문자열을 위한 청크를 보낼 수 있다.
이를 통해 각 청크의 ``Content-Length`` 헤더를 생성한다. 이 방식으로 서버는 원하는 경우에
클라이언트 연결을 유지할 수 있다. 서버가 이 방법을 사용할 때는 반드시 RFC 2616 방식을 완전히 따라야 한다.
아니면 ``Content-Length``\ 가 없을 때 사용할 수 있는 다른 방법을 사용해야 한다.

(주의: 어플리케이션과 미들웨어는 출력에 청크화나 gzip화 등 어떤 종류의 ``Transfer-Encoding``\ 도
적용해선 안된다. "홉 바이 홉" 연산에서 이 인코딩은 실제 웹 서버/게이트웨이의 영역이다.
`기타 HTTP 기능`_\ 에서 자세한 정보를 본다.)


버퍼링과 스트리밍
-----------------

일반적으로 어플리케이션은 적절한 사이즈로 출력을 버퍼링하고 한번에 보낼 때 가장 좋은 성능을 낸다.
이 방식은 Zope과 같은 기존 프레임 워크에서 일반적으로 사용되는 방식이다.
출력은 StringIO나 유사한 객체로 버퍼링되고 요청 헤더를 따라 한번에 보내진다.

WSGI에서는 리스트와 같은 단일 요소 반복 가능 객체를 반환함으로써 이를 구현한다.
이 반복 가능 객체는 단일 바이트 문자열을 응답 바디로 포함한다. 텍스트를 메모리에
쉽게 넣을 수 있는 HTML 페이지를 렌더링하는 어플리케이션 함수의 경우 대부분 이 방법을 권장한다.

하지만 큰 파일이나 멀티파트 "서버 푸쉬"와 같은 특별한 목적을 갖는 HTTP 스트리밍의 경우
어플리케이션은 더 작은 블럭에 출력을 제공해야 한다. 큰 파일을 메모리에 로딩하는 것을 막기 위함이다.
응답의 일부가 생성하는데 시간이 오래 걸리는 경우에도 응답의 앞부분을 먼저 보내는 것이 좋다.

이러한 상황에서 어플리케이션은 보통 출력을 블럭 바이 블럭 방식으로 출력하는
반복기(주로 생성자-반복기)를 반환한다. 이 블럭은 "서버 푸쉬"를 위한 멀티파트
바운더리나 디스크 파일의 다른 블럭을 읽는 것과 같은 시간이 걸리는 작업 전에 중단될 수 있다.

WSGI 서버, 게이트웨이, 미들웨어는 어떤 블럭의 전송도 지연해선 안된다. 블럭을 클라이언트로
완전히 전송하거나 어플리케이션이 다음 블럭을 생성하는 동안에도 전송을 계속한다는 것을 보장해야 한다.
서버/게이트웨이나 미들웨어는 다음 방법 중 하나를 통해 이를 보장할 수 있다.

1. 어플리케이션으로의 제어를 반환하기 전에 전체 블럭을 운영 체제로 보낸다.
   (어떤 O/S 버퍼가 플러쉬 되도록 요청한다.)

2. 다른 스레드를 사용해 어플리케이션이 다음 블럭을 생성하는 동안에
   블럭이 계속 전송된다는 것을 보장한다.

3. 전체 블럭을 부모 게이트웨이/서버로 보낸다. (미들웨어만 해당.)

위 사항들을 보장함으로써 어플리케이션은 출력 데이터의 전송이 임의의 지점에
정체되지 않는다는 것을 보장할 수 있다. 이는 멀티 파트 "서버 푸쉬" 스트리밍과 같은
것들이 적절한 기능을 하는데 있어 중요하다. 멀티 파트 바운더리 간의 데이터를
클라이언트에게 완전히 전송해야 하는 경우가 이에 해당한다.


블럭 바운더리의 미들웨어 관리
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

비동기화 어플리케이션과 서버를 잘 지원하기 위해 미들웨어 컴포넌트는
어플리케이션 반복 가능 객체로부터 여러 값을 기다리는 반복을 중단해선 안된다.
미들웨어가 어떤 출력을 생성할 수 있게 될 때까지 어플리케이션으로부터
더 많은 데이터를 축적해야 한다면 미들웨어는 반드시 빈 바이트 문자열을 반환해야 한다.

이 요구 사항을 다르게 표현하면 밑단의 어플리케이션이 값을 반환할 때마다
미들웨어는 반드시 하나 이상의 값을 반환해야 한다는 의미다. 만약 미들웨어가 다른 값을
반환할 수 없다면 빈 바이트 문자열을 반환해야 한다.

이 요구 사항을 통해 비동기화 어플리케이션과 서버가 주어진 수의 어플리케이션
인스턴스를 동시에 실행하기 위해 필요한 스레드 수를 함께 줄인다는 것을 보장한다.

이 요구사항은 다른 의미로 밑단의 어플리케이션이 반복 가능 객체를 반환하면
미들웨어는 반드시 반복 가능 객체를 바로 반환해야 한다는 의미다.
또한 미들웨어가 ``write()`` 호출 가능 객체를 사용해 밑단의 어플리케이션이
반환한 데이터를 보내는 것은 금지되어 있다. 미들웨어는 내제된 어플리케이션이
미들웨어가 제공한 ``write()`` 호출 가능 객체를 사용해 보낸 데이터를 보낼 때
부모 서버의 ``write()`` 호출 가능 객체만을 사용할 수 있다.


``write()`` 호출 가능 객체
~~~~~~~~~~~~~~~~~~~~~~~~~~

기존 어플리케이션 프레임 워크 API 중 일부는 WSGI와 다른 방식으로
버퍼되지 않은 출력을 지원한다. 이들은 "write" 함수나 버퍼되지 않은
데이터 블럭을 작성하는 메서드를 제공하거나 버퍼된 "write" 함수와
"플러쉬" 메커니즘을 사용해 버퍼를 플러쉬한다.

안타깝게도 이러한 API들은 스레드나 다른 특별한 메커니즘을 사용하지 않는 한
WSGI의 반복 가능 객체 어플리케이션 반환 값으로 구현할 수 없다.

그러므로 이러한 프레임 워크들이 계속 명령형 API를 사용할 수 있게 하려면
WSGI는 ``start_response``\ 이 반환한 특별한 ``write()`` 호출 가능 객체를 포함해야 한다.

새로운 WSGI 어플리케이션과 프레임 워크는 가능한 경우 ``write()`` 호출 가능 객체를
사용해선 안된다. ``write()`` 호출 가능 객체는 명령형 스트리밍 API를 지원하기 위한
핵으로 사용되기 때문이다. 일반적으로 어플리케이션은 자신이 반환한 반복 가능 객체를 통해
출력을 생성해야 한다. 이를 통해 웹 서버는 같은 파이썬 스레드의 다른 작업을 인터리브해
잠재적으로 서버 전체에 더 나은 처리량을 제공할 수 있다.

``start_response()`` 호출 가능 객체에 의해 반환된 ``write()`` 호출 가능 객체는 단일 매개
변수를 받는다. 이 매개 변수는 HTTP 응답 바디를 구성하기 위해 작성된 바이트 문자열로 출력 반복
가능 객체에 의해 반환된 것처럼 취급된다. 다시 말해, ``write()``\ 는 반환하기 전에 전달된
바이트 문자열이 클라이언트로 완전히 보내졌거나 어플리케이션이 진행되는 동안 전송이 버퍼되었다고 보장해야 한다.

어플리케이션은 응답 바디의 일부나 전체를 ``write()``\ 를 사용해 생성하는 경우에도
반복 가능 객체를 반드시 반환해야 한다. 반환된 객체는 비어있을 수 있다. (예시: 비어있지 않은
바이트 문자열을 반환하지 않는 경우.) 하지만 비어있지 않은 바이트 문자열을 반환한다면
그 출력은 반드시 서버나 게이트웨이에 의해 정상적으로 처리되어야 한다. (예시: 출력은 반드시 바로
보내지거나 대기되어야 한다.) 어플리케이션은 자신이 반환한 반복 가능 객체 안에서 ``write()``\ 를
호출해선 안된다 따라서 반복 가능 객체가 반환한 모든 바이트 문자열은 ``write()``\ 로
보내진 모든 바이트 문자열이 클라이언트로 보내진 이후에 반환된다.


유니코드 문제
-------------

HTTP는 유니코드와 그 인터페이스를 직접 지원하지 않는다.
모든 인코딩/디코딩은 어플리케이션으로 관리해야 한다. 서버가 보내거나 받는
모든 스트링은 ``str`` 또는 ``bytes`` 타입이어야 하고 ``unicode``\ 가 되면 안된다.
문자열 객체가 있어야 할 곳에 ``unicode`` 객체가 사용되면 정의하지 못한다.

상태나 응답 헤더로서 ``start_response()``\ 에 보내지는 문자열은 반드시
인코딩에 대해 RFC 2616를 따라야 한다. 이는 ISO-8859-1 문자이거나
RFC 2047 MIME 인코딩을 사용해야 한다는 의미다.

``str``\ 과 ``StringType`` 타입이 실제로 유니코드 베이스인 파이썬 플랫폼
(자이썬, 아이언파이썬, 파이썬3 등)에서 이 사양이 다루는 모든 "문자열"은
ISO-8859-1 인코딩으로 나타낼 수 있는 코드 포인트를 포함해야 한다.
(``\u00FF``, ``\u0000`` 포함.) 다른 유니코드 문자나 코드 포인트를 포함하는
문자열을 제공하는 것은 어플리케이션에 있어 심각한 에러다. 유사하게
서버와 게이트웨이는 다른 유니코드 문자를 포함하는 문자열을 어플리케이션에 보내선 안된다.

다시 말해, 이 사양이 "문자열"로 다루는 모든 객체는 반드시 ``str``\ 이나 ``StringType``
타입이 되어야 하고 ``unicode``\ 나 ``UnicodeType`` 타입이 되어선 안된다.
주어진 플랫폼이 ``str``/``StringType`` 객체에 문자당 8비트를 넘게 허용하더라도
이 사양의 "문자열"은 문자당 8비트 이하만을 사용해야 한다.

이 사양이 "바이트 문자열"이라 다루는 값은 파이썬 3에서는 ``bytes``,
이전 버전의 파이썬에서는 ``str``\ 이 되어야 한다.
(예시: ``wsgi.input``\ 로부터 읽는 값, ``write()``\ 로 보내지는 값,
어플리케이션이 반환하는 값.)


에러 처리
---------

일반적으로 어플리케이션은 자신의 내부 에러를 포착하고 브라우저에 도움이 되는
메시지를 보여줘야 한다. (여기서 도움이 되는 메세지란 어플리케이션에 따라 다르다.)

그러나 이러한 메세지를 보여주려면 어플리케이션은 실제로 어떤 데이터를 브라우저로 보내선 안된다.
응답이 손상될 위험이 있기 때문이다. 따라서 WSGI는 어플리케이션이 에러 메세지를 보내면서
자동으로 중단될 수 있도록 하기 위한 메커니즘을 제공한다.
``start_response``\ 의 ``exc_info`` 인수가 그 메커니즘이다. 다음은 인수 사용과 관련된 예시다. ::

    try:
        # regular application code here
        status = "200 Froody"
        response_headers = [("content-type", "text/plain")]
        start_response(status, response_headers)
        return ["normal body goes here"]
    except:
        # XXX should trap runtime issues like MemoryError, KeyboardInterrupt
        #     in a separate handler before this bare 'except:'...
        status = "500 Oops"
        response_headers = [("content-type", "text/plain")]
        start_response(status, response_headers, sys.exc_info())
        return ["error body goes here"]

예외가 발생했을 때 작성된 출력이 없으면 ``start_response``\ 로의 호출은 정상적으로
반환되고 어플리케이션은 브라우저로 보내질 에러 내용을 반환한다. 그러나 작성된 출력이
이미 브라우저로 보내졌다면 ``start_response``\ 는 받은 예외를 다시 발생시킨다.
이 예외는 어플리케이션에 의해 포착되선 안되고 어플리케이션은 중단된다.
그리고 서버나 게이트웨이는 이 (심각한) 예외를 포착하고 응답을 중단한다.

서버는 어플리케이션이나 그 반환값의 반환을 중단하는 모든 예외를 포착하고 로그해야 한다.
어플리케이션 에러가 발생했을 때 일부 응답이 브라우저에 작성되었고 이미 보내진 헤더가 서버가
문제없이 수정할 수 있는 ``text/*`` 타입이라면 서버나 게이트웨이는 에러 메세지를 출력에 추가하려는 시도를 해야 한다.

몇몇 미들웨어는 추가 예외 처리 서비스를 추가하거나 어플리케이션 에러 메세지를
인터셉트해 대체하길 윈할 수 있다. 이러한 경우에 미들웨어는 ``start_response``\ 에 제공된
``exc_info``\ 를 다시 발생시키지 않고 미들웨어 특정 예외를 발생시키거나 단순히 제공받은
인수를 저장한 이후에 예외없이 반환하는 방법을 선택할 수 있다. 이렇게 되면 어플리케이션은
자신의 에러 바디 반복 가능 객체를 반환하거나 ``write()``\ 를 호출해 미들웨어가 에러 출력을
포착하고 수정하게 할 수 있다. 이러한 방법은 어플리케이션 작성자가 다음을 지키는 한 작동한다.

1. 에러 응답을 시작할 때 항상 ``exc_info``\ 를 제공한다.

2. ``exc_info``\ 가 제공되는 동안 ``start_response``\ 가 발생시킨 에러를 포착하지 않는다.


HTTP 1.1 Expect/Continue
------------------------

HTTP 1.1를 구현한 서버와 게이트웨이는 반드시 HTTP 1.1의 "expect/continue" 메커니즘을
동일하게 지원해야 한다. 지원 방법에는 여러가지가 있다.

1. ``Expect: 100-continue`` 요청을 포함하는 요청에 "100 Continue"로 응답하고 정상적으로 진행한다.

2. 요청을 정상적으로 진행하지만 어플리케이션이 입력 스트림을 처음 읽으려 시도할 때
   "100 Continue" 응답을 보내는 ``wsgi.input`` 입력 스트림을 어플리케이션과 함께 제공한다.
   읽기 요청은 반드시 클라이언트 응답 전까지 차단된 상태로 있어야 한다.

3. 서버가 expect/continue를 지원하지 않는다는 것을 클라이언트가 결정할 때까지 기다리고
   요청 바디를 스스로 직접 보낸다. (최적의 방법이 아니고 권장하지 않는다.)

이러한 행동 제한은 HTTP 1.0 요청이나 어플리케이션으로 보내지지 않는 요청에는
적용되지 않는다. HTTP 1.1 Expect/Continue에 대한 자세한 정보는 RFC 2616의 섹션 8.2.3과 10.1.1을 본다.


기타 HTTP 기능
--------------

일반적으로 서버와 게이트웨이는 간섭하지 않고 어플리케이션의 출력에 대한 완전한 제어를 허용해야 한다.
서버와 게이트웨이는 어플리케이션 응답의 실질적인 의미가 바뀌지 않는 선에서만 변경할 수 있다.
어플리케이션 개발자는 미들웨어 컴포넌트를 추가해 추가 기능을 제공할 수 있다.
따라서 서버/게이트웨이 개발자는 구현에 있어 보수적이어야 한다.
어떤 의미에서 서버는 스스로를 어플리케이션을 HTTP "원본 서버"로 갖는
HTTP "게이트웨이 서버"처럼 생각해야 한다. (이 용어의 정의는 RFC 2616 섹션 1.3을 본다.)

WSGI 서버와 어플리케이션은 HTTP를 통해 소통하지 않기 때문에 RFC 2616이
홉 바이 홉 헤더라고 부르는 방식이 WSGI 내부 커뮤니케이션에 적용되지 않는다.
WSGI 어플리케이션은 어떤 홉 바이 홉 헤더를 생성해선 안되고 [4]_ 이러한 헤더의 생성을 요구하는
HTTP 기능의 사용을 시도하거나 ``environ`` 딕셔너리에 들어온 홉 바이 홉 헤더의 내용에 의존해선 안된다.
WSGI 서버는 적용 가능한 청크 인도킹을 포함해 모든 인바운드 ``Transfer-Encoding``\ 을 디코딩하는
것과 같이 지원되는 인바운드 홉 바이 홉 헤더를 반드시 직접 처리해야 한다.

이러한 원칙을 다양한 HTTP 기능에 적용하면 서버가 캐시 검증을 ``If-None-Match``\ 와
``If-Modified-Since`` 요청 헤더 그리고 ``Last-Modified``\ 와 ``ETag`` 요청 헤더를 통해 처리할 수
있다는 것을 분명히 해야 한다. 그러나 이는 요구 사항은 아니며 어플리케이션이 이 기능을 지원하고자
한다면 자신만의 캐시 검증을 수행해야 한다. 서버/게이트웨이는 이러한 검증을 요구받지 않기 때문이다.

이와 유사하게 서버는 어플리케이션의 응답을 재인코딩하거나 전송 인코딩할 수 있지만
어플리케이션은 자신만의 적절한 컨텐츠 인코딩을 사용해야 하고 전송 인코딩을 적용해선 안된다.
클라이언트가 어플리케이션 응답의 바이트 범위를 요청할 경우 서버는 이를 전송할 수 있고
어플리케이션은 기본적으로 바이트 범위를 지원하지 않는다. 다시 말해 어플리케이션은
원하는 경우 이 기능을 직접 수행해야 한다.

이러한 어플리케이션에 대한 제한은 모든 어플리케이션이 반드시 모든 HTTP 기능을 재구현해야 한다는
의미는 아니다. 많은 HTTP 기능은 미들웨어 컴포넌트에 의해 부분적으로 또는 완전히 구현될 수 있다.
따라서 서버와 어플리케이션 작성자 모두 같은 기능을 다시 구현하지 않아도 된다.


스레드 지원
-----------

스레드 지원은 서버에 따라 다르다. 다수의 요청을 병렬로 실행할 수 있는 서버는
어플리케이션 실행을 단일 스레드 방식으로 실행하는 옵션을 제공해야 한다.
이를 통해, 스레드 안전하지 않은 어플리케이션이나 프레임 워크를 서버에서 사용할 수 있게 된다.



구현/어플리케이션 주의 사항
===========================


서버 확장 API
-------------

서버 작성자가 어플리케이션, 프레임 워크 작성자의 특별한 목적을 위한 고급 API를 제공하길 원할 수 있다.
예를 들어, ``mod_python``\ 에 기반한 게이트웨이는 아파치(Apache) API의 일부를 WSGI 확장으로 제공하려 한다.

가장 간단한 경우에는 ``mod_python.some_api``\ 와 같은 ``environ`` 변수를 정의하면 된다.
하지만 많은 경우에 미들웨어의 유무로 인해 문제가 어려워진다. 예를 들어, ``environ`` 변수에
나타나는 동일한 HTTP 헤더로의 접근을 제공하는 API는 미들웨어에 의해 ``environ`` 변수가
수정되어 다른 데이터를 반환할 수 있다.

일반적으로 WSGI 기능의 일부를 복제, 대체, 우회하는 모든 확장 API는 미들웨어 컴포넌트와 호환되지 않을 수 있다.
서버/게이트웨이 개발자는 누군가 미들웨어를 사용할 수 있다는 것을 가정해야 한다.
프레임 워크 개발자는 프레임 워크가 다양한 미들웨어의 기능을 하도록 구성하거나 재구성할 수 있기 때문이다.

따라서 서버와 게이트웨이가 WSGI 기능을 대체하는 확장 API를 제공할 때는 반드시 이 API가 대체하는 API의 일부를
사용해 호출되도록 설계해 호환성을 최대한 제공해야 한다. 예를 들어, HTTP 요청 헤더로 접근하기 위한 확장 API는
어플리케이션이 현재 ``environ`` 안에서 보내지게 해야 한다. 이를 통해 서버/게이트웨이는 HTTP 헤더가 미들웨어에
의해 수정되지 않은 API를 통해 접근 가능함을 확인할 수 있다. 확장 API가 ``environ``\ 과 일치하는 HTTP 헤더 내용을
갖는다고 보장하지 못하면 에러를 발생시키는 것과 같은 방법을 통해 어플리케이션으로의 서비스를 거부하고 헤더 모음 대신
``None`` 값이나 API에 적합한 어떤 값을 반환해야 한다.

유사하게 확장 API가 응답 데이터나 헤더를 작성하는 다른 방법을 제공하면 어플리케이션이 추가 서비스를
얻기 전에 ``start_response``\ 호출 가능 객체를 받을 수 있게 해야한다. 만약 받은 객체가 서버/게이트웨이가
기존에 어플리케이션에 제공하던 것과 다른 것이면 제대로 처리된다고 보장할 수 없고 어플리케이션에 추가 서비스를
제공하는 것을 거부해야 한다.

이 가이드라인은 파싱된 쿠키, 형태 변수, 세션 등의 정보를 ``environ``\ 에 추가하는 미들웨어에 적용된다.
이러한 미들웨어는 단순히 값을 ``environ``\ 에 넣어선 안되고 ``environ``\ 을 처리하는 함수로 이 기능을 제공해야 한다.
이를 통해 어떤 미들웨어가 URL을 재작성하거나 다른 ``environ`` 수정을 한 후에 ``environ``\ 으로부터
정보를 계산한다고 것을 보장할 수 있다.

이러한 "안전 확장" 규칙을 서버/게이트웨이 개발자와 미들웨어 개발자가 지키는 것이 중요하다.
미래의 미들웨어 개발자가 확장 API를 사용하는 어플리케이션으로 인한 혼란을 피하기 위해
일부 또는 모든 확장 API를 ``environ``\ 에서 삭제하는 상황을 막기 위함이다.


어플리케이션 설정
-------------------------

이 사양은 서버가 불러올 어플리케이션을 선택하거나 얻는 방식을 정의하지 않는다.
이 옵션이나 다른 설정 옵션은 서버 특성에 많이 의존한다.
서버/게이트웨이 작성자는 서버가 특정 어플리케이션 객체를 실행하도록 설정하는 방법과
스레딩 옵션 등 어떤 옵션을 사용할지에 대해 문서화할 수 있다.

반대로 프레임 워크 작성자는 프레임 워크의 기능을 둘러싸는 어플리케이션 객체를
생성하는 방법에 대해 문서화해야 한다. 서버와 어플리케이션 프레임 워크를 모두 선택한
사용자는 반드시 이 둘을 연결해야 하지만 서버와 프레임 워크 모두 공통된 인터페이스를
갖기 때문에 새로운 서버/프레임 워크 쌍을 위해 큰 노력을 들이진 않아도 된다.

마지막으로 몇몇 어플리케이션, 프레임 워크, 미들웨어에서 간단한 설정 옵션 문자열을
받기 위해 ``environ`` 딕셔너리를 사용하길 원할 수 있다. 서버와 게이트웨이는 어플리케이션
배포자가 이름-값 쌍이 ``environ``\ 에 위치하도록 지정하는 것을 허용해 이 방식을 지원해야 한다.
가장 간단한 경우에는 ``os.environ``\ 에서 운영 체제가 제공하는 모든 환경 변수를
``environ`` 딕셔너리에 복사함으로써 지원할 수 있다. 원칙적으로 배포자는 외부에서 이 환경 변수를
서버에 설정할 수 있기 때문이다. CGI의 경우에는 서버의 설정 파일을 통해 이 변수를 설정할 수 있다.

모든 서버가 변수의 간편한 설정을 지원하진 않기 때문에 어플리케이션은
이러한 필수 변수를 최소한으로 유지해야 한다. 물론 최악의 경우에도
어플리케이션을 배포하는 사람은 스크립트를 생성해 필요한 설정값을 제공할 수 있다. ::

   from the_app import application

   def new_app(environ, start_response):
       environ['the_app.configval1'] = 'something'
       return application(environ, start_response)

하지만 대부분의 기존 어플리케이션과 프레임 워크는 ``environ``\ 으로부터
어플리케이션이나 프레임 워크 특정 설정 파일의 위치를 알려주는 설정값 하나만을 필요로 할 것이다.
(물론 어플리케이션은 설정값을 캐시해 호출할 때마다 이 파일을 다시 읽지 않도록 해야 한다.)


URL 재작성
------------------

어플리케이션이 리퀘스트의 전체 URL을 재작성하려 할 때 Ian Bicking이 만든 다음 알고리즘을 사용할 수 있다. ::

    from urllib.parse import quote
    url = environ['wsgi.url_scheme']+'://'

    if environ.get('HTTP_HOST'):
        url += environ['HTTP_HOST']
    else:
        url += environ['SERVER_NAME']

        if environ['wsgi.url_scheme'] == 'https':
            if environ['SERVER_PORT'] != '443':
               url += ':' + environ['SERVER_PORT']
        else:
            if environ['SERVER_PORT'] != '80':
               url += ':' + environ['SERVER_PORT']

    url += quote(environ.get('SCRIPT_NAME', ''))
    url += quote(environ.get('PATH_INFO', ''))
    if environ.get('QUERY_STRING'):
        url += '?' + environ['QUERY_STRING']

재작성된 URL은 클라이언트가 요청한 URI와 완전히 일치하지 않을 수 있다는 것을 알아두자.
예를 들어, 서버의 재작성 규칙이 클라이언트가 요청한 URL을 표준 형식에 맞게 고칠 수 있다.


2.2 이전의 파이썬 버전 지원
------------------------------------------

Some servers, gateways, or applications may wish to support older
(<2.2) versions of Python.  This is especially important if Jython
is a target platform, since as of this writing a production-ready
version of Jython 2.2 is not yet available.

For servers and gateways, this is relatively straightforward:
servers and gateways targeting pre-2.2 versions of Python must
simply restrict themselves to using only a standard "for" loop to
iterate over any iterable returned by an application.  This is the
only way to ensure source-level compatibility with both the pre-2.2
iterator protocol (discussed further below) and "today's" iterator
protocol (see PEP 234).

(Note that this technique necessarily applies only to servers,
gateways, or middleware that are written in Python.  Discussion of
how to use iterator protocol(s) correctly from other languages is
outside the scope of this PEP.)

For applications, supporting pre-2.2 versions of Python is slightly
more complex:

* You may not return a file object and expect it to work as an iterable,
  since before Python 2.2, files were not iterable.  (In general, you
  shouldn't do this anyway, because it will perform quite poorly most
  of the time!)  Use ``wsgi.file_wrapper`` or an application-specific
  file wrapper class.  (See `선택 플랫폼 특정 파일 관리`_
  for more on ``wsgi.file_wrapper``, and an example class you can use
  to wrap a file as an iterable.)

* If you return a custom iterable, it **must** implement the pre-2.2
  iterator protocol.  That is, provide a ``__getitem__`` method that
  accepts an integer key, and raises ``IndexError`` when exhausted.
  (Note that built-in sequence types are also acceptable, since they
  also implement this protocol.)

Finally, middleware that wishes to support pre-2.2 versions of Python,
and iterates over application return values or itself returns an
iterable (or both), must follow the appropriate recommendations above.

(Note: It should go without saying that to support pre-2.2 versions
of Python, any server, gateway, application, or middleware must also
use only language features available in the target version, use
1 and 0 instead of ``True`` and ``False``, etc.)


선택 플랫폼 특정 파일 관리
----------------------------------------

몇몇 운영 환경은 특별한 고성능 파일 전송 기능을 제공한다. 유닉스의 ``sendfile()`` 호출을 예로 들 수 있다.
서버와 게이트웨이는 이 기능을 ``environ``\ 의 선택적 ``wsgi.file_wrapper`` 키로 이러한 기능을 표시할 수 있다.
어플리케이션은 이 "파일 래퍼"를 사용해 파일이나 파일형 객체를 반복 가능 객체로 반환할 수 있다. 예시 ::

    if 'wsgi.file_wrapper' in environ:
        return environ['wsgi.file_wrapper'](filelike, block_size)
    else:
        return iter(lambda: filelike.read(block_size), '')

만약 서버나 게이트웨이가 ``wsgi.file_wrapper``\ 를 제공하면 필수 위치 매개 변수와 선택 위치 매개 변수를 하나씩 받는 호출 가능 객체가 되어야 한다.
첫번째 매개 변수는 보내질 파일형 객체이고 두번째 매개 변수는 선택 블럭 사이즈 "제안"이다.
서버/게이트웨이는 두번째 매개 변수를 사용하지 않아도 된다. 호출 가능 객체는 반드시 반복 가능 객체를 반환해야 하고
서버/게이트웨이가 실제로 어플리케이션의 반환값으로 반복 가능 객체를 받지 않는 한 데이터를 전송해선 안된다.
(그렇지 않으면 미들웨어가 응답 데이터를 해석하거나 덮어쓰지 못하게 된다.)

어플리케이션이 제공한 객체가 "파일형" 객체가 되려면 선택 사이즈 인수를 받는 ``read()`` 메서드를 반드시 가져야 한다.
``close()`` 메서드도 가질 수 있지만 이 경우에 ``wsgi.file_wrapper``\ 가 반환한 반복 가능 객체는 기존 파일형 객체의
``close()`` 메서드를 호출하는 ``close()`` 메서드를 가져야 한다. 만약 "파일형" 객체가 파이썬 빌트인
객체(예시: ``fileno()``)와 같은 이름의 메서드나 속성을 가지면 ``wsgi.file_wrapper``\ 는 이 메서드나 속성이 빌드틴 파일
객체와 같은 의미를 갖는다고 가정할 것이다.

어떤 플랫폼 특정 파일 관리의 실제 구현은 반드시 어플리케이션이 반환한 후에 이루어져야 하고
서버나 게이트웨이는 래퍼 객체가 반환되었는지 확인해야 한다. (다시 말해, 미들웨어, 에러 처리 등으로 인해
생성된 어떤 래퍼도 실제로 사용된다고 보장할 수 없다.)

``close()`` 메서드 처리를 떠나서 어플리케이션으로부터의 파일 래퍼 반환은 의미적으로 어플리케이션이
``iter(filelike.read, '')``\ 를 반환하는 것과 같아야 한다. 이는 전송이 "파일" 내부에 있는 현재 위치에서
시작해야 하고 파일이 끝나거나 ``Content-Length`` 바이트 만큼 작성될 때까지 계속되어야 한다는 의미다.
(어플리케이션이 ``Content-Length``\ 를 제공하지 않으면 서버는 밑단의 파일을 구현하는데 사용된 것을 참고해
``Content-Length``\ 를 생성할 수 있다.)

물론 플랫폼 특정 파일 전송 API는 보통 임의의 "파일형" 객체를 받지 않는다.
따라서 ``wsgi.file_wrapper``\ 는 ``fileno()``\ (유닉스형 OS)나 ``java.nio.FileChannel``\ (Jython) 같은 객체를 위해
반환된 객체를 인트로스펙션해 파일형 객체가 플랫폼 특정 API와 함께 쓰기에 적합한지 결정해야 한다.

객체가 플랫폼 API에 적합하지 않더라도 ``wsgi.file_wrapper``\ 는 반드시 ``read()``\ 와 ``close()``\ 를
래핑하는 반복 가능 객체를 반환해 파일 래퍼를 사용하는 어플리케이션이 플랫폼 간에 이식 가능하게 해야한다.
다음은 이전(2.2 이하) 파이썬과 최신 파이썬 모두에 적합하고 간단한 플랫폼 독립 파일 래퍼 클래스다.  ::

    class FileWrapper:

        def __init__(self, filelike, blksize=8192):
            self.filelike = filelike
            self.blksize = blksize
            if hasattr(filelike, 'close'):
                self.close = filelike.close

        def __getitem__(self, key):
            data = self.filelike.read(self.blksize)
            if data:
                return data
            raise IndexError

다음은 이 래퍼 클래스로 플랫폼 특정 API에 접근을 제공하는 서버/게이트웨이에서 가져온 코드다. ::

    environ['wsgi.file_wrapper'] = FileWrapper
    result = application(environ, start_response)

    try:
        if isinstance(result, FileWrapper):
            # check if result.filelike is usable w/platform-specific
            # API, and if so, use that API to transmit the result.
            # If not, fall through to normal iterable handling
            # loop below.

        for data in result:
            # etc.

    finally:
        if hasattr(result, 'close'):
            result.close()


질문과 답변
=====================

1. ``environ``\ 이 딕셔너리여야 하는 이유는 무엇인가? 서브 클래스를 사용하면 안되는 이유는?

   딕셔너리를 요구하는 이유는 이식성을 최대한 좋게 하기 위함이다.
   딕셔너리의 메서드를 서브셋으로 정의해 표준이면서 이식할 수 있는 인터페이스가 되도록 하는게
   대안이 될 수 있다. 하지만 실제로 대부분의 서버가 필요에 맞는 딕셔너리를 선택하고 없는 것보다 낫기 때문에
   프리엠 워크 작성자는 딕셔너리 전체 기능을 기대한다. 하지만 일부 서버가 딕셔너리를
   사용하지 않기로 한다면 서버 자체의 사양 적합성과 관계없이 다른 서버와 상호 호환성 문제가 발생할 수 있다.
   따라서 딕셔너리 지원을 강제하는 것이 사양을 간편하게 하고 상호 호환성을 보장할 수 있다.

   딕셔너리를 강제한다고 해서 서버나 프레임 워크 작성자가 ``environ`` 딕셔너리의 커스텀 변수로
   특화된 서비스를 제공할 수 없는 것은 아니다. 이러한 값을 추가한 서비스를 제공하기 위해 권장하는 방법이다.

2. ``write()``\ 를 호출해 바이트 스트링이나 반복 가능 객체를 반환할 수 있는 이유는 무엇인가?
   한가지 방법만 사용해야 하는 것은 아닌가?

   반복만 지원하게 되면 "푸쉬" 지원을 가정하는 현재 프레임 워크에 문제가 생긴다.
   반대로 ``write()``\ 를 통해 "푸쉬"만 지원하면 큰 파일을 전송할 때 서버 성능에 문제가 생긴다.
   (워커 스레드가 모든 출력을 보내야만 새 요청에 대한 작업을 시작할 수 있는 경우.)
   두가지 방법을 모두 허용함으로써 서버 구현자는 푸쉬만 지원하는 경우보다 부담을 덜 수 있다.

3. ``close()``\ 의 목적은?

   어플리케이션 객체를 실행하는 동안 작성이 완료되면 어플리케이션은 리소스가 try/finally 블럭을
   사용해 해제된다고 보장할 수 있다. 하지만 어플리케이션이 반복 가능 객체를 반환하면 이 객체가
   쓰레기 수집이 될 때까지 사용된 리소스가 해제되지 않는다. ``close()``\ 는 어플리케이션이
   요청 끝에 중요한 리소스를 해제할 수 있게 하고 향후에 PEP325에 제안된 대로 생성자에서 try/finally을 지원할 수 있다.

4. 인터페이스가 너무 로우 레벨인 것은 아닌가? 나는 어떤 기능을 원한다. (쿠키, 세션, 지속성)

   이것은 또 다른 파이썬 웹 프레임 워크가 아니다. 프레임 워크가 서버와 통신하거나 그 반대를 위한 방법일 뿐이다.
   어떤 기능을 원한다면 그 기능을 지원하는 웹 프레임 워크를 선택해야 한다. 그리고 그 프레임 워크로 WSGI
   어플리케이션 생성이 가능하다면 이를 대부분의 WSGI 지원 서버에서 실행할 수 있어야 한다. 또한 몇몇 WSGI
   서버는 ``environ`` 딕셔너리에 제공된 객체를 통해 추가 서비스를 지원할 수 있다. 세부 사항은 적용 가능한
   서버 문서를 본다. (물론 이러한 확장을 사용하는 어플리케이션은 다른 WSGI 기반 서버에 이식 불가능해진다.)

5. 오래되고 좋은 HTTP 헤더 대신 CGI 변수를 사용하는 이유는 무엇인가?
   그리고 이 변수를 WSGI 정의 변수와 합치는 이유는?

   많은 기존 웹 프레임 워크는 CGI 사양에 크게 의존하여 빌드되었고 기존 웹 서버는 CGI 변수를 생성하는
   방법을 알고 있다. 반대로 인바운드 HTTP 정보를 표시하는 대안적인 방법은 단편화 되었고 시장 점유율이 떨어진다.
   따라서 CGI "표준"을 사용하는 것은 기존 구현을 활용하는 좋은 방법으로 보인다. 이 변수들은 WSGI 변수와
   나누게 되면 두개의 딕셔너리 인수가 필요해질 뿐 실제로 얻는 이득은 없다.

6. 상태 문자열을 ``"200 OK"`` 대신 ``200``\ 만 사용하면 안되는 것인가?

   서버와 게이트웨이 입장에서는 숫자 상태와 메세지를 담고 있는 테이블을 가져야 하므로 복잡해진다.
   반대로 어플리케이션이나 프레임 워크 작성자에게 특정 응답 코드에 맞는 추가 텍스트를 작성하는 것은 쉽다.
   보통 기존 프레임 워크는 메시지가 담긴 테이블을 이미 갖고 있다. 따라서 서버/게이트웨이에 맞추는 것이 낫다.

7. ``wsgi.run_once``\ 가 어플리케이션을 한번만 실행한다고 보장할 수 없는 이유는?

   ``wsgi.run_once``\ 는 자주 실행되지 않게 설정해야 하는 응용프로그램에 대한 제안일 뿐이다.
   캐싱, 세션 등을 위한 여러 실행 모드를 갖는 어플리케이션 프레임 워크를 위해 고안되었다.
   "다중 실행" 모드에서 이러한 프레임 워크는 캐시를 미리 로드하고 각 요청 이후에 디스크로 로그나
   세션 데이터를 작성하지 않을 수 있다. "단일 실행" 모드에서는 프레임 워크가 미리 로드하는 것을
   방지하고 각 요청 이후에 필요한 작성을 플러쉬할 수 있다.

   그러나 어플리케이션이나 프레임 워크가 단일 실행 모드에서 올바르게 실행되는지 테스트하기 위해
   한번 이상 호출해보는 것이 좋다. 따라서 ``wsgi.run_once``\ 를 ``True``\ 로 설정했다고 해서
   어플리케이션이 딱 한번만 실행된다고 가정하면 안된다.

8. 어떤 기능(딕셔너리, 호출 가능 객체 등)이 어플리케이션 코드에서 사용하기 좋지 않다.
   대신 객체를 사용하면 안되는가?

   WSGI의 모든 구현 결정은 기능들을 분리하기 위함이다. 이 기능들을 압축된 객체에 다시 합치는 것은 서버나
   게이트웨이 작성을 어렵게 한다. 또한 전체 기능 중 아주 일부만 수정하거나 대체하는 미들웨어 작성도 어렵게 한다.

   다른 함수는 그대로 두면서 어떤 함수를 위한 "핸들러"로 작용할 때 미들웨어는 기본적으로
   "책임 사슬" 패턴을 가지려 한다. 인터페이스가 확장 가능한 채로 남으려 하면 일반적인 파이썬 객체로는
   작업하기 어려워진다. 예를 들어, 확장(향후의 WSGI 버전이 정의한 속성 등)이 보내진다는 것을 보장하려면
   사용자는 반드시 ``__getattr__``\ 나 ``__getattribute__`` 덮어쓰기를 사용해야 한다.

   이러한 타입의 코드는 100% 맞기 아주 어렵고 대부분 직접 작성하려 하지 않는다.
   따라서 다른 사람이 구현한 것을 복사하지만 이 사람이 다른 부분을 수정하면 업데이트하지 못한다.

   추가로 이 필수적인 표준안은 어플리케이션 프레임 워크 개발자를 위해 조금 더 보기 좋은 API를
   지원하기 위해 미들웨어 개발자가 내는 세금 같은 것이다. 하지만 어플리케이션 프레임 워크 개발자는 보통
   자신의 프레임 워크 중 극히 일부에서만 WSGI를 지원하기 위해 하나의 프레임 워크만을 업데이트한다.
   보통 이들은 WSGI 구현을 처음이자 마지막으로 하기 때문에 사용할 사양만을 구현할 것이다. 따라서
   객체 속성으로 API를 보기 좋게 만드는 노력은 낭비가 된다.

   더 보기 좋거나 개선된 WSGI 인터페이스를 웹 프레임 워크 개발과는 반대로 웹 어플리케이션 프로그래밍에
   직접 사용하길 원할 수 있다. 이들에게는 어플리케이션 개발자가 간편하게 사용할 수 있도록 WSGI를 래핑하는
   API나 프레임 워크를 개발하는 것을 권장한다. 이렇게 하면 WSGI는 서버와 미들웨어 작성자가 편리하게 작업할 수
   있을 만큼 로우 레벨로 남으면서 어플리케이션 개발자 입장에서도 나쁘게 보이지 않을 것이다.


제안되었거나 진행중인 토의
=========================

이 항목들은 Web-SIG나 다른 것에서 현재 토의 중이거나 PEP 작성자의 "할 일" 목록에 있는 것들이다.

* ``wsgi.input``\ 이 파일 대신 반복자여야 하는가?
  이 방식은 비동기화 어플리케이션과 청크 인코딩 입력 스트림에 도움이 된다.

* 입력이 있거나 콜백이 발생할 때까지 어플리케이션 출력 반복을 중단하는 선택 확장에 대해 토의하고 있다.

* 동기화와 비동기화 어플리케이션과 서버, 스레딩 모델 관련 사항, 이 영역에 대한 문제와 설계 목표 추가.


감사 인사
================

Web-SIG 메일링 리스트의 사려 깊은 피드백을 주어 이 문서를 작성할 수 있게 해준 분들께 감사한다. 특히:

* Gregory "Grisha" Trubetskoy, author of ``mod_python``, who beat up
  on the first draft as not offering any advantages over "plain old
  CGI", thus encouraging me to look for a better approach.

* Ian Bicking, who helped nag me into properly specifying the
  multithreading and multiprocess options, as well as badgering me to
  provide a mechanism for servers to supply custom extension data to
  an application.

* Tony Lownds, who came up with the concept of a ``start_response``
  function that took the status and headers, returning a ``write``
  function.  His input also guided the design of the exception handling
  facilities, especially in the area of allowing for middleware that
  overrides application error messages.

* Alan Kennedy, whose courageous attempts to implement WSGI-on-Jython
  (well before the spec was finalized) helped to shape the "supporting
  older versions of Python" section, as well as the optional
  ``wsgi.file_wrapper`` facility, and some of the early bytes/unicode
  decisions.

* Mark Nottingham, who reviewed the spec extensively for issues with
  HTTP RFC compliance, especially with regard to HTTP/1.1 features that
  I didn't even know existed until he pointed them out.

* Graham Dumpleton, who worked tirelessly (even in the face of my laziness
  and stupidity) to get some sort of Python 3 version of WSGI out, who
  proposed the "native strings" vs. "byte strings" concept, and thoughtfully
  wrestled through a great many HTTP, ``wsgi.input``, and other
  amendments.  Most, if not all, of the credit for this new PEP
  belongs to him.


참조
==========

.. [1] "웹 프로그래밍"에 대한 파이썬 위키
   (https://wiki.python.org/moin/WebProgramming)

.. [2] 일반 게이트웨이 인터페이스 사양, v 1.1, 3rd Draft
   (https://tools.ietf.org/html/draft-coar-cgi-v11-03)

.. [3] "청크 전송 코딩" -- HTTP/1.1, section 3.6.1
   (http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.6.1)

.. [4] "엔드 투 엔드와 홉 바이 홉 헤더" -- HTTP/1.1, Section 13.5.1
   (http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.5.1)

.. [5] mod_ssl 레퍼런스, "환경 변수"
   (http://www.modssl.org/docs/2.8/ssl_reference.html#ToC25)

.. [6] PEP \333 수정 사항에 대한 절차 문제.
   (https://mail.python.org/pipermail/python-dev/2010-September/104114.html)

.. [7] PEP 333과의 차이점을 볼 수 있는 PEP \3333 SVN 리비전 기록.
   (http://svn.python.org/view/peps/trunk/pep-3333.txt?r1=84854&r2=HEAD)

저작권
=========

이 문서는 퍼블릭 도매인에 속한다.



..
   Local Variables:
   mode: indented-text
   indent-tabs-mode: nil
   sentence-end-double-space: t
   fill-column: 70
   End:
