

# Week 2, Create Model



- 머신러닝 모델을 만드는 전체 과정은 아래 세 가지 단계로 분리하여 생각 할 수 있다.

  1. 데이터 셋 생성
  2. 모델의 구조 생성
  3. 모델을 운용하는 것

  이 중 이번 세션에서는 모델의 구조 생성하는 방법에 대해 학습한다.



- **Estimator API**
  - 학습 모델 정의, 모델 학습 및 평가 과정들을 모듈화 해놓은 TenserFlow High Level API이다.  Estimator API 작업은 아래와 같이 두 부분으로 나눌 수 있다.
    - 머신러닝 모델을 설정하는 부분
      1. 회귀인가 분류인가?
      2. 레이블로 사용될 필드는 무엇인가?
      3. 사용 될 Feature 목록은 무엇인가?
    - 머신러닝을 수행하는 부분
      1. 모델을 학습 시키는 것
      2. 모델을 평가하는 것
      3. 예측 결과를 뽑아내 보는 것
  - ![image-20190624152802632](/Users/iseungcheon/Library/Application Support/typora-user-images/image-20190624152802632.png)

- **Wide and Deep**

  - wide 한 모델은 로지스틱 회귀 모델을 이용하여 추천 알고리즘을 작성하여 학습을 시킨 경우, 학습 데이타를 기반으로 상세화된 예측 결과를 리턴해준다. 예를 들어 검색 키워드 (프라이드 치킨)으로 검색한 사용자가 (치킨과 와플)을 주문한 기록이 많았다면, 이 모델은 (프라이드 치킨)으로 검색한 사용자는 항상 (치킨과 와플)을 추천해주게 된다.  즉 예전에 기억된 값 (Memorization된 값)을 통해서 예측을 하는데, 이러한 모델을 와이드 모델이라고 한다.
  - 뉴럴네트워크 모델의 경우 프라이드 치킨을 햄버거, 프랜치 프라이등을 일반화 시켜서 패스트 푸드로 분류하여 프라이드 치킨으로 검색을 해도 이와 같은 종류의 햄버거를 추천해도 사용자가 택할 가능성이 높다. 이러한 모델을 딥모델이라고 하는데, 딥 모델의 경우 문제점이, 너무 일반화가(under fitting)  되서 엉뚱한 결과가 나올 수 있다는 것인데, 예를 들어서 따뜻한 아메리카노를 검색했는데, 커피라는 일반화 범주에서 아이스 라떼를 추천해줄 수 있다는 것이다. 즉 커피라는 일반화 범주에서 라떼는 맞는 추천일 수 있지만, 따뜻한 음료를 원하는 사람에게 차가운 음료를 추천하는 지나친 일반화가 발생할 수 있다.

- Category data Incoding**

  - ML model은 모든 feature들을 숫자로 기억하고 있다. 그러나 입력 데이터 형식은 numberical(숫자형)과 categorical(범주형)의 데이터가 있고 categorical 데이터에 대해서는 별도의 인코딩과정이 필요하다.

  - **feature column** 은 **raw data** 와 **Estimator**사이의 **중재자 역할**로, Estimator가 raw data를 사용할 수 있도록 다양한 형식들을 제공한다.

  - ```categorical_column_with_vocabulary_list``` 는 **key, vocabulary_list** 두개의 인자를 받고, vocabulary_list값들을 인코딩해준다. 주로 범주형 컬럼이 가질 수 있는 모든 값을 미리 알고 있을 경우 사용한다.

  - ```indicator_column``` 는 **one hot - incoding** 하는데 사용된다.

  - 숫자형 데이터 입력데이터에 대해서도 ```numeric_column``` 를 이용하여 실제 모델에서도 숫자형 데이터로 사용 될 수 있게 설정해주어야 한다.

    ```python
    # Define feature columns
    def get_categorical(name, values):
      return tf.feature_column.indicator_column(
       tf.feature_column.categorical_column_with_vocabulary_list(name, values))
    
    def get_cols():
      # Define column types
      return [
              get_categorical('is_male', ['True', 'False', 'Unknown']),
              tf.feature_column.numeric_column('mother_age'),
              get_categorical('plurality',
                          ['Single(1)', 'Twins(2)', 'Triplets(3)',
                           'Quadruplets(4)', 'Quintuplets(5)','Multiple(2+)']),
              tf.feature_column.numeric_column('gestation_weeks')
          ]
    ```

- **input Function**

  ```python
  # Create an input function reading a file using the Dataset API
  # Then provide the results to the Estimator API
  def read_dataset(filename, mode, batch_size = 512):
    def _input_fn():
      def decode_csv(value_column):
        '''
        	value_column : csv파일의 한 row
        	return : value_column에서 label을 분리하고, 딕셔너리형태로 label과 리턴
        '''
        columns = tf.decode_csv(value_column, record_defaults=DEFAULTS) # value_column을 csv 파일로 인식하여 , 를 기준으로 쪼개어 배열에 담는다.
        features = dict(zip(CSV_COLUMNS, columns)) # 딕셔너리 생성
        label = features.pop(LABEL_COLUMN)
        return features, label
      
      # Create list of files that match pattern
      # Glob는 인자로 넣어준 정규식과 일치하는 파일리스트를 가져온다
      file_list = tf.gfile.Glob(filename) 
  
      # Create dataset from file list
      #  TextLineDataSet을 통해 각 개행마다 배열로
      dataset = (tf.data.TextLineDataset(file_list)  # Read text file
                   .map(decode_csv))  # Transform each elem by applying decode_csv fn
        
      if mode == tf.estimator.ModeKeys.TRAIN:
          num_epochs = None # indefinitely
          dataset = dataset.shuffle(buffer_size=10*batch_size)
      else:
          num_epochs = 1 # end-of-input after this
   
      dataset = dataset.repeat(num_epochs).batch(batch_size)
      return dataset
    return _input_fn
  ```

- 학습 및 평가

- ```python
  def train_and_evaluate(output_dir):
    EVAL_INTERVAL = 300
    # 체크포인트 저장 콜백 함수 설정, 300초마다 콜백함수가 호출되고 체크포인트는 최대 3개까지 저장
    run_config = tf.estimator.RunConfig(save_checkpoints_secs = EVAL_INTERVAL,
                                        keep_checkpoint_max = 3)
    
   # model_dir은 최종모델이 저장될 경로, feature_columns로는 입력될 데이터들의 형식 리스트, hidden_units은 레이어의 뉴런 수를 담는 리스트
    estimator = tf.estimator.DNNRegressor(
                         model_dir = output_dir,
                         feature_columns = get_cols(),
                         hidden_units = [64, 32],
                         config = run_config)
   # 분산 학습을 위해 Train spce 및 Eval spce 정의
    train_spec = tf.estimator.TrainSpec(
                         input_fn = read_dataset('train.csv', mode = tf.estimator.ModeKeys.TRAIN),
                         max_steps = TRAIN_STEPS)
  # exporter는 그래프 및 체크포인트를 별도로 저장하는 방식을 지정하는 객체로 EvalSpec이 평가내용을 저장할 때 참조된다.
    exporter = tf.estimator.LatestExporter('exporter', serving_input_fn)
  # Eval_spec 정의, throttle_secs마다 평가 과정을 수행
    eval_spec = tf.estimator.EvalSpec(
                         input_fn = read_dataset('eval.csv', mode = tf.estimator.ModeKeys.EVAL),
                         steps = None,
                         start_delay_secs = 60, # start evaluating after N seconds
                         throttle_secs = EVAL_INTERVAL,  # evaluate every N seconds
                         exporters = exporter)
    # 실제 학습 및 검증 수행을 실행하는 함수 호출
    tf.estimator.train_and_evaluate(estimator, train_spec, eval_spec)
  ```

- 실행

  ```python
  # Run the model
  
  # babyweight_trained 디렉토리 트리를 몽땅 삭제
  shutil.rmtree('babyweight_trained', ignore_errors = True) 
  # 텐서보드 이벤트 파일을 위해서, FileWriter 캐시를 비워둠
  tf.summary.FileWriterCache.clear() 
  # ensure filewriter cache is clear for TensorBoard events file
  train_and_evaluate('babyweight_trained')
  ```

  

- TenserFlow 모니터링

  ```python
  from google.datalab.ml import TensorBoard
  TensorBoard().start('./babyweight_trained')
  ```

- wide는 강의자료로 대신한다.

  - ![image-20190624153904374](/Users/iseungcheon/Library/Application Support/typora-user-images/image-20190624153904374.png)

  - ![image-20190624153919955](/Users/iseungcheon/Library/Application Support/typora-user-images/image-20190624153919955.png)