# Extract, Analyze, and Translate Text from Images with the Cloud ML APIs

- 이번 세션에서는 구글 클라우드에서 이미지에서 텍스트를 추출하고, 해당 텍스트를 번역 및 분석하는 작업에 대해 학습한다.
- TEXT_DETECTION에 관한 보다 자세한 사항은 [여기](https://cloud.google.com/vision/docs/ocr)를 참고

## 0. 버킷에 이미지 저장

- 버킷 생성 과정은 매우 간단하기에 생략한다.
- 버킷의 이름은 유일해야 한다.
- 단 이미지를 올린 뒤 반드시 권한을 확인한다. 모든 접근자에 대해 읽기 권한이 허용되어 있어야 한다.

## 1.  request.json 생성 후 API 요청

- 'ocr-request.json'이라는 이름으로 아래와 같은 내용을 포함하는 파일을 생성한다.
- 'gscImageUrl' 의 value는 본인이 만든 "gs://버킷이름/파일이름" 으로 사용한다.

        {
          "requests": [
              {
                "image": {
                  "source": {
                      "gcsImageUri": "gs://my-bucket-name/sign.jpg"
                  }
                },
                "features": [
                  {
                    "type": "TEXT_DETECTION",
                    "maxResults": 10
                  }
                ]
              }
          ]
        }

- curl 명령어를 통해 아래와 같이 google vision api에 요청한다.

        curl -s -X POST -H "Content-Type: application/json" --data-binary @ocr-request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY} -o ocr-response.json
        
        # -o 를 통해 response 값을 저장 할 수 있다.

- **API_KEY** 는 구글 클라우드에 발급받은 키를 사전에 환경변수로 등록하여 사용한다.
- curl로 요청이 성공적으로 이루어졌다면 아래와 같이 사진에서 추출한 텍스트들에 대한 정보가 response가 반환된다.

        {
          "responses": [
            {
              "textAnnotations": [
                {
                  "locale": "fr",
                  "description": "LE BIEN PUBLIC\nles dépeches\nPour Obama,\nla moutarde\nest\nde Dijon\n",
                  "boundingPoly": {
                    "vertices": [
                      {
                        "x": 146,
                        "y": 48
                      },
                      {
                        "x": 621,
                        "y": 48
                      },
                      {
                        "x": 621,
                        "y": 795
                      },
                      {
                        "x": 146,
                        "y": 795
                      }
                    ]
                  }
                },
        
                    ...
              ]
        }]
        }

## 3. 텍스트 번역

- translation api에 요청할 'translation-request.json' 파일을 만든다.

        {
          "q": "your_text_here",
          "target": "en"
        }

- [jq](https://www.lesstif.com/pages/viewpage.action?pageId=42074200) 명령어를 통해 'q'의 value를 수정한다. [sed](https://hyunkie.tistory.com/51)는 편집 명령어이다

        STR=$(jq .responses[0].textAnnotations[0].description ocr-response.json) && STR="${STR//\"}" && sed -i "s|your_text_here|$STR|g" translation-request.json

- translation api에 요청

        curl -s -X POST -H "Content-Type: application/json" --data-binary @translation-request.json https://translation.googleapis.com/language/translate/v2?key=${API_KEY} -o translation-response.json

- translation-response.json 파일 확인, 아래와 같이 번역된 문자와, 번역이 진행된 언어의 정보가 담겨있다. 구글 클라우드 API는 101개가 넘은 언어를 지원한다. 지원 목록은 [여기](https://cloud.google.com/translate/docs/languages)를 참고

        {
          "data": {
            "translations": [
              {
                "translatedText": "THE PUBLIC GOOD the despatches For Obama, the mustard is from Dijon",
                "detectedSourceLanguage": "fr"
              }
            ]
          }
        }

자연어 API는 **DOC 참고**