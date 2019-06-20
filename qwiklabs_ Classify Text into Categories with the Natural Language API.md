### 

- 학습할 내용

  + 자연어 API 요청 생성
  + 자연어 API를 통한 글자분류
  + 글자분류를 통한 기사 이해

- 진행하기 전에

  + API 키 발급 및 환경변수 설정

    ```bash
    export API_KEY = [API 키]
    ```

    

- 뉴스 분류

  1. request 생성

     ```json
     # request.json
     {
       "document":{
         "type":"PLAIN_TEXT",
         "content":"A Smoky Lobster Salad With a Tapa Twist. This spin on the Spanish pulpo a la gallega skips the octopus, but keeps the sea salt, olive oil, pimentón and boiled potatoes."
       }
     }
     ```

  2. API 요청

     ```bash
     curl "https://language.googleapis.com/v1/documents:classifyText?key=${API_KEY}" \
       -s -X POST -H "Content-Type: application/json" --data-binary @request.json
     ```

     ```json
     # retrun
     { categories:
       [
         {
           name: '/Food & Drink/Cooking & Recipes',
            confidence: 0.85
         },
         {
            name: '/Food & Drink/Food/Meat & Seafood',
            confidence: 0.63
          }
       ]
     }
     
     ```

     요청을 보낸 본문에는 Food라는 단어가 존재하지 않음에도, NL API는 문맥을 파악하여 글의 내용을 추출해냈음을 알 수 있다.

  3. **Classify a large text dataset**

     - .

       ```bash
       gsutil cat gs://text-classification-codelab/bbc_dataset/entertainment/001.txt
       ```

     - BigQuery 생성

       - 좌측 네이게이션바에서 BigQuery 선택
       - 좌측 박스에 GCP Project ID 클릭
       - news_classification_dataset 이름으로 데이터셋 생성
       - 아래와 같은 옵션으로 테이블 생성
         - Create From: **empty table**
         - Name your table **article_data**
         - Click **Add Field** and add the following 3 fields: article_text, category, and confidence(FLOAT).

     - service account 생성

       ```bash
       export PROJECT=<your_project_name>
       ```

       ```bash
       gcloud iam service-accounts create my-account --display-name my-account
       gcloud projects add-iam-policy-binding $PROJECT --member=serviceAccount:my-account@$PROJECT.iam.gserviceaccount.com --role=roles/bigquery.admin
       gcloud iam service-accounts keys create key.json --iam-account=my-account@$PROJECT.iam.gserviceaccount.com
       export GOOGLE_APPLICATION_CREDENTIALS=key.json
       ```

     -  `classify-text.py` 작성 후 실행(2분정도 걸림)

       ```python
       from google.cloud import storage, language, bigquery
       
       # Set up our GCS, NL, and BigQuery clients
       storage_client = storage.Client()
       nl_client = language.LanguageServiceClient()
       # TODO: replace YOUR_PROJECT with your project id below
       bq_client = bigquery.Client(project='YOUR_PROJECT')
       
       dataset_ref = bq_client.dataset('news_classification_dataset')
       dataset = bigquery.Dataset(dataset_ref)
       table_ref = dataset.table('article_data') # Update this if you used a different table name
       table = bq_client.get_table(table_ref)
       
       # Send article text to the NL API's classifyText method
       def classify_text(article):
               response = nl_client.classify_text(
                       document=language.types.Document(
                               content=article,
                               type=language.enums.Document.Type.PLAIN_TEXT
                       )
               )
               return response
       
       rows_for_bq = []
       files = storage_client.bucket('text-classification-codelab').list_blobs()
       print("Got article files from GCS, sending them to the NL API (this will take ~2 minutes)...")
       
       # Send files to the NL API and save the result to send to BigQuery
       for file in files:
               if file.name.endswith('txt'):
                       article_text = file.download_as_string()
                       nl_response = classify_text(article_text)
                       if len(nl_response.categories) > 0:
                               rows_for_bq.append((article_text, nl_response.categories[0].name, nl_response.categories[0].confidence))
       
       print("Writing NL API article data to BigQuery...")
       # Write article text + category data to BQ
       errors = bq_client.insert_rows(table, rows_for_bq)
       assert errors == []
       ```

     - 이후 BigQuery 탭에서 select 쿼리문을 날리면 해당 데이터들과 분류된 카테고리 값을 확인 할 수 있다.

