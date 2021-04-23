---
title:  "[etc]라인으로 알람을 받아보자"
excerpt: "line 메신저를 통해알람을 받아보자"

categories:
  - etc
tags:
  - [etc, Programming]

toc: true
toc_sticky: true
 
date: 2019-12-02
last_modified_at: 2019-12-02
---

*개인이 무료로 이용가능하고 이미지와 함께 알람 까지 받기 위해서 찾아봤을때 Line Notify 가 제일 괜찮아 보였음

---

# LINE Notify
### 공식 홈피 : https://notify-bot.line.me/en/
### API 문서 : https://notify-bot.line.me/doc/en/

# 시작해보자
## Step 1
* 라인 메신저에 이메일/PW  등록하기
    (이 정보로 Line Notify서비스 이용)

## Step 2
* access token 생성하기
    https://notify-bot.line.me/my/

## Step3
* 알람을 발송할때 사용할 이름과 대화방 지정하기
* 생성하면 아래처럼 라인으로 축하메세지가 와요.

## Step4
아래 API 가이드에 따라 코드를 만들어 보자



## Step5
메세지만 보내기 성공
이미지도 함께 보내기 성공



# 샘플코드
## 메시지만 손쉽게 보내기 위해서 
* `Content-Type:application/x-www-form-urlencoded` 사용하는 간략화한 샘플코드
```C++
size_t CallbackRecvHeader( void* recvp, size_t size, size_t nmemb, void* userp )
{
	size_t realsize = size * nmemb;
	if(realsize <= 0)
		return 0;

	std::string* headermessage = (std::string*)userp;
	headermessage->insert( headermessage->size(), (char*)recvp );

	return size * nmemb;
}

size_t CallbackWriteMemory( void* recvp, size_t size, size_t nmemb, void* userp )
{
	size_t realsize = size * nmemb;
	if(realsize <= 0)
		return 0;

	std::string* result = (std::string*)userp;
	result->insert( result->size(), (char*)recvp );

	return size * nmemb;
}
.
.
.
CURL* curl_ = curl_easy_init();
curl_slist* requestHeaders_ = NULL;
std::string result_;
std::string httpHeaderData_;

requestHeaders_ = curl_slist_append(requestHeaders_, "Content-Type:application/x-www-form-urlencoded; charset=UTF-8");
requestHeaders_ = curl_slist_append(requestHeaders_, "Authorization: Bearer 토큰");

std::string bodyString = "message=testmessage";

curl_easy_setopt(curl_, CURLOPT_URL, "https://notify-api.line.me/api/notify");
curl_easy_setopt(curl_, CURLOPT_TIMEOUT, 3);
curl_easy_setopt(curl_, CURLOPT_HTTPHEADER, requestHeaders_);	
curl_easy_setopt(curl_, CURLOPT_WRITEFUNCTION, CallbackWriteMemory);
curl_easy_setopt(curl_, CURLOPT_WRITEDATA, &result_);
curl_easy_setopt(curl_, CURLOPT_HEADERFUNCTION, CallbackRecvHeader);
curl_easy_setopt(curl_, CURLOPT_HEADERDATA, &httpHeaderData_);
curl_easy_setopt(curl_, CURLOPT_POSTFIELDS, bodyString.c_str() );
curl_easy_setopt(curl_, CURLOPT_POSTFIELDSIZE, bodyString.length() );
curl_easy_setopt(curl_, CURLOPT_SSL_VERIFYPEER, false);
curl_easy_setopt(curl_, CURLOPT_SSL_VERIFYHOST, false);

curl_easy_perform(curl_);

curl_slist_free_all(requestHeaders_);
curl_easy_cleanup(curl_);
```


## 메세지와 이미지도 보내기 위해서
* `Content-Type:multipart/form-data` 사용하는 간략화한 샘플코드
* buff에 png나 jpeg 데이터가 있다고 가정
```c++
CURL* curl_;
CURLcode res;
std::string result_;

struct curl_httppost* post = NULL;
struct curl_httppost* last = NULL;

curl_slist* requestHeaders_;

curl_ = curl_easy_init();
if (curl_)
{
	requestHeaders_ = NULL;
	requestHeaders_ = curl_slist_append(requestHeaders_, "Content-Type:multipart/form-data");
	requestHeaders_ = curl_slist_append(requestHeaders_, "Authorization: Bearer 토큰");
	CURLcode error_code_ = curl_easy_setopt(curl_, CURLOPT_HTTPHEADER, requestHeaders_);

	curl_formadd(&post, &last,
		CURLFORM_COPYNAME, "imageFile",
		CURLFORM_BUFFER, "imageFile",
		CURLFORM_BUFFERPTR, &buff[0],
		CURLFORM_BUFFERLENGTH, buff.size(),
		CURLFORM_END);

	curl_formadd(&post, &last,
		CURLFORM_COPYNAME, "message",
		CURLFORM_COPYCONTENTS, "image-test-data",
		CURLFORM_END);

	curl_easy_setopt(curl_, CURLOPT_URL, "https://notify-api.line.me/api/notify");
	curl_easy_setopt(curl_, CURLOPT_HTTPPOST, post);


	error_code_ = curl_easy_setopt(curl_, CURLOPT_WRITEFUNCTION, CallbackWriteMemory);
	if (CURLE_OK != error_code_)
	{
		return false;
	}

	error_code_ = curl_easy_setopt(curl_, CURLOPT_SSL_VERIFYPEER, false);
	if (CURLE_OK != error_code_)
	{
		return false;
	}

	error_code_ = curl_easy_setopt(curl_, CURLOPT_SSL_VERIFYHOST, false);
	if (CURLE_OK != error_code_)
	{
		return false;
	}

	error_code_ = curl_easy_setopt(curl_, CURLOPT_WRITEDATA, &result_);
	if (CURLE_OK != error_code_)
	{
		return false;
	}

	res = curl_easy_perform(curl_);
	if (res)
	{
		return 0;
	}

	printf("%s\n", result_.c_str());

	curl_slist_free_all(requestHeaders_);
	curl_formfree(post);
}
else
{
	return 0;
}

curl_easy_cleanup(curl_);
}
```



# API Limit
* API limit 가 토큰당 1000번
* image 를 보낼때에는 시간당 제한이따로 있음
* 현재 상태를 알기 위해선 아래가이드처럼 Http 응답 헤더를 보면 됨
* 응답 헤더 정보를 보면 아래처럼 되어있음
```
HTTP/1.1 200
Server: nginx
Date: Tue, 03 Dec 2019 00:33:47 GMT
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Keep-Alive: timeout=3
X-RateLimit-Limit: 1000
X-RateLimit-ImageLimit: 50
X-RateLimit-Remaining: 997
X-RateLimit-ImageRemaining: 50
X-RateLimit-Reset: 1575336631
```

# 에필로그
* 가이드가 한글빼고 다른언어로 웬만큼 되어있어.. 라인은 역시 한국에선 듣보잡 서비스인가?
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2019-12-02-etc-line-001.png)

* API Limit 가 콘솔상으로 확인하지 못함 반면 구글은 아래처럼 해줌 구글 만세!
    * 구글 API (http://webholic.kr/music 에서 사용하는 youtube 검색 API 사용 관련 콘솔) 샘플
	![image]({{ site.url }}{{ site.baseurl }}/assets/images/2019-12-02-etc-line-002.png)
	![image]({{ site.url }}{{ site.baseurl }}/assets/images/2019-12-02-etc-line-003.png)