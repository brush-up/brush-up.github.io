---
title:  "AWS DDB의 데이터를 지표화 해보도록 노력해보기. DDB -> S3 -> Athena"
excerpt: "AWS DDB의 데이터를 지표화 해보도록 노력해보기. DDB -> S3 -> Athena"

categories:
  - etc
tags:
  - [AWS, Programming]

toc: true
toc_sticky: true
 
date: 2023-12-11
last_modified_at: 2023-12-11

published: true
---


# Intro
* AWS 로 지표관련을 진행하면서 알게된 사항에 대해 정리해 보기.
* AWS 콘솔이 은근 바뀌어서 이제는 다를 수도 있지만, 일단은 여기까지만.

# 1. DDB 에서 S3 로 데이터 이전하기
1. IAM role 생성 필요.
* 이 부분은 잘 모르니 pass
    * 아마 Permissions Policies section 의 AWSGlueServiceRoe, AmazonDynamoDBFullAccess, AmazonS3FullAccess 등이 필요한듯?

2. DDB 에서 S3 로 데이터 추출하기
* step1. Glue Database 생성하기.
    * AWS Glue console 로 이동
    * Data Catalog 에서 Database 선택후 Add database 로 추가하자
    * {database_name}_ddb 와 같은 식으로 이름을 지어본뒤 Create database 로 생성하자. 

* step2. Glue Crawler 생성하기 (source 는 DDB, destination 은 Glue)
    * AWS Glue console 에서 
    * Data Catalog 에서 Crawlers 선택후 Create crawler 로 추가하자
    * {tableName_ddb_glue_crawler} 와 같은 크롤러 이름으로 만들어 보자 ( {tableNAme}은 S3 으로 내보내려는 DynamoDB 의 테이블 이름으로 해보자. )
    * 데이터가 아직 Glue 테이블에 매핑되지 않았으므로 Add a data source 를 실행하자.
    * Drop-down 메뉴에서 Data souce 를 DynamoDB 를 선택하자. 
    * S3 로 내보내려는 DynamoDB 의 테이블 이름을 지정하자
    * Glue, S3, DynamoDB 에 접근하기 위해 이전에 생성한 IAM role 을 선택하자. 
    * target database 로 이전에 생성한 Glue Database 를 선택하자 
    * Crawler schedule 를 On-demand 로 설정하자. (추후 스케줄을 바꿀예정)
    * Create crawler 로 crawler 를 생성하자

* step3. Glue table 생성을 위해 Crawler 를 수행하기.
    * AWS Glue → Data Catalog → Crawlers
    * step2 에서 Run crawler 로 생성했던 crawler 를 run 하자. 
    * complete 되면 Glue table 이 생성이 된다.
    * AWS Glue → Data Catalog → Databases
    * 원하는 table 을  database 에서 선택한뒤
    * list 에서 최근 생성된 항목을 선택해서 스키마 정보가 이상없는지 확인해보자. 

* step4. S3 Bucket 생성하기.

* step5. scheduled Glue ETL 생성하기 (소스는 Glue, destination 은 S3)
    * AWS Glue → ETL Jobs 
    * Visual with a source and target 를 선택한뒤
    * Source 는 AWS Glue Data Catalog 를 선택하고, Target 은 Amazon S3 로 선택하자
    * Data source
        * Data catalog 선택후 이름을 지정하고
        * DynamoDB source 에서 Choose from the AWS Glue Data Catalog 를 선택하자 
            * 많은 예시들이 Choose the DynamoDB table directly 를 선택해서 진행하는데, 이렇게 하면 기본적으로 ddb 의 pk, sk 등만 옮겨진다, 일일이 하나하나 지정해줘야지 다른 컬럼들을 복사할수가 있다. 
        * Database 에서 이전에 생성한 것으로 지정하자. 
        * Table 에서 이전한 테이블 이름을 선택하고.
    * Transfrom
        * S3 로 내보내고 싶지 않는 항목이 있으면 여기서 제거하자. 
            * 최근에 업데이트로 자동으로 선택되지 않는듯하다. 수동으로 선택해서 추가하자.
                * Change Schema 를 추가하자?
    * Data Target
        * Format(JSON), Compression Type 등을 지정하자
            * Format, Compression Type 별 차이는 모르겠....
                * https://medium.com/@autumn.bom/aws-athena-convert-to-parquet-with-snappy-89c93af3c8c0 이 자료보면 도움이 좀 될듯하다. 
    * 네비게이션 바에서 Job detail 을 클릭해서 IAM role 등을 설정 후 
    * 꼭 Save 후 Run 을 하자. (이건 step6. 이후 진행하자. ) 
        * 현재 화면에 나와있는 세팅으로 Run 되지 않는다. 꼭 Save 하자. 
        * job 이 실행완료가 되면 위에서 설정한 S3 에 데이터가 잘 보일 것이다. 

* step6. Script 수정하기.
    * batch 가 실행되는 시점의 날짜별로 파티션을 나누기 위해 아래 작업을 진행함

```
import datetime
.
.
# 현재 날짜 가져오기
current_date = datetime.datetime.now().strftime('%Y-%m-%d-%H')
print(f"Current date is: {current_date}")
glueContext.get_logger().info(f"Current date is: {current_date}")
 
 
year, month, day, hour = current_date.split("-")
print(f"Current date year is: {year}")
glueContext.get_logger().info(f"Current date year is: {year}")
.
.
//ApplyMapping 에 아래 추가.
        ("year", "string", year, "string"),
        ("month", "string", month, "string"),
        ("day", "string", day, "string"),
        ("hour", "string", hour, "string"),
.
.
.
// 아래와 같은 형식으로 s3 path 지정,
    connection_options={
        "path": f"s3://.../.../ddb-to-s3/year={year}/month={month}/day={day}/hour={hour}/",
        "partitionKeys": [],
    },
```

* 스크립트 수정후 run 하자. 
    * 실행한 이후엔 항상 s3 용 glue crawler 를 돌려야함.!!

* https://docs.aws.amazon.com/ko_kr/glue/latest/dg/incremental-crawls.html 이런 글이 있는데, 이거 더 확인해봐야 할듯. 
* 참고로, 이제는 쿼리 날릴때 꼭 where 문에 year, month, hour 등을 넣어야함



# 2. S3 에서 Athena 로 Query 하기. 

* step1. Glue Database 생성하기
    * AWS Glue → Data Catalog → Databases 에서 Add database
    * {databseName}_s3 같은 이름을 지은후 Create 하자.
* step2. glue crawler 생성하기 (source 는 S3, destination 은 Glue Table)
    * AWS Glue → Data Catalog → Crawlers 에서 create Crawler 하기
    * {tableNAme}_s3_glue_crawler 같은 이름을 짓기
    * Data source configuration 에서 Not yet 으로 설정후 data source 추가하기.
    * Drop down 으로  Data source 는 S3로 지정후 데이터를 읽을 S3 path 등을 지정하자. 
    * IAM role 지정하고 Target database 는 위에서 생성한 Glue Database 로 하고 crawler schedule 는 Ondemand 로 지정후 Create crawler 로 하면!
* step3. Crawler 를 실행해 Glue table 생성하기
    * AWS Glue → Data Catalog → Crawlers 에서 생성했던 crawler 를 선택해서 Run 시키자. 
    * crawler 가 한번 성공적으로 runs 되면 glue table 가 생성될 것이다. (스키마 확인해 보기)
    * AWS Glue → Data Catalog → Databases 에서 database 를 클릭해서 최근생선된 테이블을 확인해 보자. 
* step4. Query 결과를 담을 S3. 생성하기.
    * Athena 에서 query 한 결과물을 받을 s3 를 따로 만들자. 
* step5. crawler 에 의해  생성한 Glue table 을 Athena 로 쿼리하기.
    * 네비게이션 바에서 
        * Data source 는 AwsDataCatalog 로 하고
        * Database 는 위에서 {databaseNAme}_s3 등으로 생선한 것으로 고르고. 
    * 쿼리를 때리면 실행이 될 것이다


# 3. DDB 에서 S3 로 데이터 좀더 잘 이전하기 
* 위의 예시에서 DDB 를 S3로 이전할때 URI가 항상 동일한 문제가 있다. 여러가지로 확인해본 결과 visual ETL로는 되지 않는것을 확인


* Step1.
    * AWS Glue → ETL jobs → Visual ETL 에서 생성했던 job 을 선택하자. 
* Step2. 
    * Visual 옆의 Script 를 선택해서 로직을 추가하자. 
    * Script 를 건들면 Visual 은 사라져 버린다.. 참고하세요.
    * 아래와 같은 로직으로 추가해서 실행할때마다 S3 path 를 다르게 지정할수 있다. 

```
import datetime
...
# Script generated for node S3 bucket
 
current_datetime = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
output_path = f"s3://.../.../ddb-to-s3/{current_datetime}/"
 
S3bucket_node3 = glueContext.write_dynamic_frame.from_options(
    frame=ApplyMapping_node2,
    connection_type="s3",
    format="json",
    connection_options={
#        "path": "s3://.../.../ddb-to-s3/",   # 기존 로직.
        "path": output_path,
        "partitionKeys": [],
    },
    transformation_ctx="S3bucket_node3",
)
 
job.commit()
```

# 4. 특정 주기로 Query 해보기.
* 몇몇 가지 방법이 있는듯 하지만. SDK 를 이용해서 하는 법이 나에겐 가장 용이해 보임
    * https://repost.aws/knowledge-center/schedule-query-athena
* 괜찮은 설명이 없어 우선 Java SDK를 이용해서 Athena 로 Query 하는거 진행 해봄.
* step1.
    * Athena 에 접근하기 위한 client 생성
* step2.
    * query 실행해서 execution ID 받아오기
* step3.
    * query 실행이 완료될때까지 기다려서 결과값 가져오기

```
// https://docs.aws.amazon.com/athena/latest/ug/code-samples.html 이거 참고해서 구현중.
    int CLIENT_EXECUTION_TIMEOUT = 100000;
    String ATHENA_OUTPUT_BUCKET = "s3://output/athena-test-results/";
    String ATHENA_SAMPLE_QUERY = "select * from \"test_origin_ddb\".\"ddb_to_s3\" limit 10;"; // change the Query statement to match your environment
    long SLEEP_AMOUNT_IN_MS = 1000;
    String ATHENA_DEFAULT_DATABASE = "test_origin_ddb"; // change the database to match your database
 
    public void test_athena(){
        log.info("\n\n ============================= test_athena start =============================\n\n");
 
 
        String accessKeyId = ".......";
        String secretAccessKey = "........";
        String secToken = "......";
 
        AwsCredentialsProvider awsCredentialsProvider = StaticCredentialsProvider.create(AwsSessionCredentials.create(accessKeyId, secretAccessKey, secToken));     
        AthenaClient athenaClient = AthenaClient.builder()
            .region(Region.AP_NORTHEAST_2)
            .credentialsProvider(awsCredentialsProvider)
            .httpClient(UrlConnectionHttpClient.builder().build())
            .build();
 
        String queryExecutionId = submitAthenaQuery(athenaClient);
        try {
            waitForQueryToComplete(athenaClient, queryExecutionId);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        processResultRows(athenaClient, queryExecutionId);
        athenaClient.close();           
    }
 
    public String submitAthenaQuery(AthenaClient athenaClient) {
        try {
            // The QueryExecutionContext allows us to set the database.
            QueryExecutionContext queryExecutionContext = QueryExecutionContext.builder()
                .database(ATHENA_DEFAULT_DATABASE)
                .build();
 
            // The result configuration specifies where the results of the query should go.
            ResultConfiguration resultConfiguration = ResultConfiguration.builder()
                .outputLocation(ATHENA_OUTPUT_BUCKET)
                .build();
 
            StartQueryExecutionRequest startQueryExecutionRequest = StartQueryExecutionRequest.builder()
                .queryString(ATHENA_SAMPLE_QUERY)
                .queryExecutionContext(queryExecutionContext)
                .resultConfiguration(resultConfiguration)
                .build();
 
            StartQueryExecutionResponse startQueryExecutionResponse = athenaClient.startQueryExecution(startQueryExecutionRequest);
            return startQueryExecutionResponse.queryExecutionId();
 
        } catch (AthenaException e) {
            e.printStackTrace();
            System.exit(1);
        }
        return "";
    }   
    // Wait for an Amazon Athena query to complete, fail or to be cancelled.
    public void waitForQueryToComplete(AthenaClient athenaClient, String queryExecutionId) throws InterruptedException {
        GetQueryExecutionRequest getQueryExecutionRequest = GetQueryExecutionRequest.builder()
            .queryExecutionId(queryExecutionId)
            .build();
 
        GetQueryExecutionResponse getQueryExecutionResponse;
        boolean isQueryStillRunning = true;
        while (isQueryStillRunning) {
            getQueryExecutionResponse = athenaClient.getQueryExecution(getQueryExecutionRequest);
            String queryState = getQueryExecutionResponse.queryExecution().status().state().toString();
            if (queryState.equals(QueryExecutionState.FAILED.toString())) {
                throw new RuntimeException("The Amazon Athena query failed to run with error message: " + getQueryExecutionResponse
                        .queryExecution().status().stateChangeReason());
            } else if (queryState.equals(QueryExecutionState.CANCELLED.toString())) {
                throw new RuntimeException("The Amazon Athena query was cancelled.");
            } else if (queryState.equals(QueryExecutionState.SUCCEEDED.toString())) {
                isQueryStillRunning = false;
            } else {
                // Sleep an amount of time before retrying again.
                Thread.sleep(SLEEP_AMOUNT_IN_MS);
            }
            System.out.println("The current status is: " + queryState);
        }
    }
    // This code retrieves the results of a query
    public void processResultRows(AthenaClient athenaClient, String queryExecutionId) {
        try {
 
            // Max Results can be set but if its not set,
            // it will choose the maximum page size.
            GetQueryResultsRequest getQueryResultsRequest = GetQueryResultsRequest.builder()
                .queryExecutionId(queryExecutionId)
                .build();
 
            GetQueryResultsIterable getQueryResultsResults = athenaClient.getQueryResultsPaginator(getQueryResultsRequest);
            for (GetQueryResultsResponse result : getQueryResultsResults) {
                List<ColumnInfo> columnInfoList = result.resultSet().resultSetMetadata().columnInfo();
                List<Row> results = result.resultSet().rows();
                processRow(results, columnInfoList);
            }
 
        } catch (AthenaException e) {
            e.printStackTrace();
            System.exit(1);
        }
    }
    private void processRow(List<Row> row, List<ColumnInfo> columnInfoList) {
        for (Row myRow : row) {
            List<Datum> allData = myRow.data();
            for (Datum data : allData) {
                System.out.println("The value of the column is "+data.varCharValue());
            }
        }
    }
```




* 현재 우리가 사용하는 개발환경에 맞게 lamda + golang 으로도 해보기
    * lamda. 제한이 5분 이긴 하지만. 
        * https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html
    * long-running worker 접근에 따라 
        * https://docs.aws.amazon.com/step-functions/latest/dg/concepts-activities.html
    * 장기실행 작업자 접근법으로 가능은 하는듯.
        * https://docs.aws.amazon.com/step-functions/latest/dg/amazon-states-language-task-state.html#amazon-states-language-task-state-activity


```
package echo
 
import (
    "fmt"
    "net/http"
    "ororchat-api/plLogger"
    "time"
 
    "github.com/aws/aws-sdk-go/aws"
    "github.com/aws/aws-sdk-go/aws/session"
    "github.com/aws/aws-sdk-go/service/athena"
    "github.com/labstack/echo/v4"
)
 
// https://gist.github.com/kartiksura/93160be1078648a14ec0ddc125c35546
func (handler AdminEchoInputAdaptor) AdminAthneaTest(ctx echo.Context) error {
 
    fmt.Println("--------------")
 
    plLogger.Error(ctx.Request().Context(), "AdminAthneaTest Start")
 
    awscfg := &aws.Config{}
    awscfg.WithRegion("ap-northeast-2")
    // Create the session that the service will use.
    sess := session.Must(session.NewSession(awscfg))
 
    svc := athena.New(sess, aws.NewConfig().WithRegion("ap-northeast-2"))
    var s athena.StartQueryExecutionInput
    s.SetQueryString("select * from ddb_to_s3 limit 10")
 
    var q athena.QueryExecutionContext
    q.SetDatabase("nick_test_origin_ddb")
    s.SetQueryExecutionContext(&q)
 
    var r athena.ResultConfiguration
    r.SetOutputLocation("s3://output/athena-test-results/test001/")
    s.SetResultConfiguration(&r)
 
    result, err := svc.StartQueryExecution(&s)
    if err != nil {
        fmt.Println(err)
        return err
    }
    fmt.Println("StartQueryExecution result:")
    fmt.Println(result.GoString())
 
    var qri athena.GetQueryExecutionInput
    qri.SetQueryExecutionId(*result.QueryExecutionId)
 
    var qrop *athena.GetQueryExecutionOutput
    duration := time.Duration(2) * time.Second // Pause for 2 seconds
 
    for {
        qrop, err = svc.GetQueryExecution(&qri)
        if err != nil {
            fmt.Println(err)
            return err
        }
        if *qrop.QueryExecution.Status.State != "RUNNING" {
            break
        }
        fmt.Println("waiting.")
        time.Sleep(duration)
 
    }
    if *qrop.QueryExecution.Status.State == "SUCCEEDED" {
 
        var ip athena.GetQueryResultsInput
        ip.SetQueryExecutionId(*result.QueryExecutionId)
 
        op, err := svc.GetQueryResults(&ip)
        if err != nil {
            fmt.Println(err)
            return err
        }
        fmt.Printf("%+v", op)
    } else {
        fmt.Println(*qrop.QueryExecution.Status.State)
 
    }
 
    return ctx.JSON(http.StatusOK, "{}")
}
```

* 개발 참고
    * https://gist.github.com/kartiksura/93160be1078648a14ec0ddc125c35546
    * https://frozenpond.tistory.com/211


* 아래 자료 더 봐야함
    * https://medium.com/@autumn.bom/aws-athena-partitioning-a3ac1c40011d
    * https://velog.io/@brillog/AWS-Glue%EB%A1%9C-DynamoDB-%EB%8D%B0%EC%9D%B4%ED%84%B0-S3%EC%97%90-%EC%A0%81%EC%9E%AC%ED%95%98%EA%B8%B0-ETL
    * https://itecnote.com/tecnote/amazon-web-services-how-to-partition-data-by-datetime-in-aws-glue/



# 참고
* https://repost.aws/knowledge-center/schedule-query-athena 
* https://stackoverflow.com/questions/76221483/aws-glue-job-job-bookmark-incremental-daily-extraction-from-rds-to-s3 
* https://medium.com/@1ra4vi3/etl-pipeline-data-from-ddb-to-s3-and-query-data-in-s3-using-aws-athena-4b7b52d7c289
* https://docs.aws.amazon.com/athena/latest/ug/code-samples.html
* https://medium.com/@autumn.bom/aws-athena-convert-to-parquet-with-snappy-89c93af3c8c0
* https://medium.com/@autumn.bom/aws-athena-partitioning-a3ac1c40011d
* https://docs.aws.amazon.com/glue/latest/dg/update-from-job.html

