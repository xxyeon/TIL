# 엑셀 파일 생성 기능을 개선했다고 했는데, 만일 대용량 엑셀 파일을 생성해야할 때 서버에 발생할 수 있는 문제로는 어떤게 있을까?

## 1. 동기/블로킹 작업으로 인한 요청 지연

> 내 생각..

대량의 row를 포함한 엑셀 파일을 서버에서 동기적으로 생성하게 되면, 요청 처리 시간이 길어져 API 응답 지연이 발생할 수 있습니다.
엑셀 생성은 CPU 연산과 함께 디스크 I/O 작업이 동반되기 때문에, 서버 리소스를 많이 점유하게 되고, 그로 인해 다른 요청이 CPU를 제대로 할당받지 못하거나 큐에 쌓여 병목 현상이 발생할 수 있습니다.
특히 WAS 스레드가 블로킹되는 구조라면 전체 서비스의 응답성이 저하될 위험도 있습니다.

이를 해결할 수 있는 방법으로는 캐싱을 사용하면 좋을 것 같다고 생각합니다.
Nginx와 같은 웹서버를 리버스 프록시로 두어서 가장 최근에 생성한 엑셀 파일을 캐싱해두는 것입니다. 하지만 고객 입장에서 현재 페이지에 보이는 정보를 동기화한 엑셀 파일이 다운 받아질 것으로 예상했는데 실제로 받아보면 이전 데이터가 담긴 엑셀파일이 다운로드 되는 일을 방지하기 위해 동기화를 짦은 주기로 시행해야할 것 같습니다.

### 캐싱 전략의 불안정성

최근 엑셀을 캐시에 저장해두는 방식은 다수의 사용자 도는 자주 바뀌는 데이터에서는 적합하지 않을 수 있다. 캐시가 오래되면 정확하지 않는 데이터를 제공할 수 있기 때문이다.  
따라서 캐시는 Fallback 용으로반 사용하고, 백엔드에서는 큐/비동기 시스템으로 정확한 엑셀을 나중에 갱신해서 제공한다. Redis 등의 TTL 캐시와 Redis Pub/Sub, Kafka 이벤트 기반 업데이트를 고려할 수 있다.

## 2. 메모리 오버헤드 및 OOM(OutOfMemoryError)

Java 기반 서버에서 Apache POI, JXL 등으로 엑셀 파일 생성 시 전체 데이터를 메모리에 올리는 구조이면 OOM이 자주 발생한다. 특히 수만 ~ 수십만개 row가 있는 엑셀일 경우 심각하다.  
해결 방법으로는 스트리밍 방식 라이브러리를 사용하는 것입니다. Apache POI의 SXSSFWorkbook, 또는 StreamingReader를 사용하여 메모리 소비를 최소화할 수 있습니다.  
또는 엑셀 파일을 백그라운드 작업ㅇ로 오프로드하려 즉시 생성하지 않고, 비동기로 생성 후 다운로드 링크를 제공할 수 있습니다.

## 3. 타임아웃 및 클라이언트 이탈

대용량 파일 생성에 시간이 오래 걸릴 경우 클라이언트 요청이 HTTP 타임아웃으로 실패하게 됩니다. 리버스 프록시등의 proxy_read_timeout, WAS의 요청 제한 시간 등에 걸릴 수도 있습니다.  
해결 방법으로는 비동기 처리와 상태 폴링 방식을 도입하여 파일 생성 요청 시 즉시 응답(202 Accepted)을 주고, 생성이 완료되면 다운로드 URL응답을 하는 것입니다.  
또는 웹 소캣/알림 방식으로 "엑셀 생성 완료됨" 이벤트 알림을 구현할 수 있습니다.

## 4. 보안/파일 크기 문제

대량 파일 크기를 생성했다고 해도 파일 업로드 시 악성 코드, 잘못된 포맷, 너무 큰 파일 업로드 가능성이 있습니다. 업로드된 엑셀을 파싱하는 로직에서 보안 취약점이 발생할 수 있습니다. (XML bomb)  
해결 방법으로는 파일 크기를 제한하고, 파일 타입을 검사하는 겁니다(MIME, 확장자 모두). 또는 Excel 파일 파서에서 zip bomb 등 검증을 적용할 수 있습니다.

## 정리

대용량 엑셀을 서버에서 생성하면 메모리나 CPU 사용량이 증가해서 병목이 생기거나 OOM 에러가 날 수 있습니다.
그래서 저는 스트리밍 방식의 엑셀 생성 라이브러리나, 생성 작업을 비동기로 큐에 넣고, 완료되면 클라이언트에게 다운로드 링크를 전달하는 방식을 고민했습니다.
장애 상황에 대비해서 최근 엑셀 파일을 캐시해 두는 것도 좋지만, 데이터 정확도가 중요한 경우에는 fallback 용도로만 사용하고, 생성이 완료되면 캐시를 갱신하는 방식이 더 안정적일 것 같습니다.

## 스트리밍 방식으로 OOM 방지

### 일반 방식으로 엑셀 생성하면

```java
XSSFWorkbook workbook = new XSSFWorkbook();
```

Apache POI의 XSSFWorkbook은 엑셀 파일 전체 구조를 메모리에 올려서 작업합니다.

모든 셀, 시트, 스타일 정보 등을 RAM에 저장한 다음 → 마지막에 파일로 한 번에 써요.

단점:

- 수만~수십만 row는 금방 수백 MB 이상 메모리를 차지

- OOM(OutOfMemoryError) 발생 위험

- 사용자 응답도 느림 (파일이 다 준비될 때까지 기다려야 전송 시작)

### 스트리밍 방식을 사용하면

```java
SXSSFWorkbook workbook = new SXSSFWorkbook(100);  // 스트리밍 기반, 100행만 메모리에 유지
```

SXSSFWorkbook은 XSSFWorkbook의 스트리밍 버전입니다.

내부적으로 row들을 임시 파일에 기록하면서 메모리에 최소한만 유지해요.

기본적으로 최근 N개의 row만 메모리에 올리고, 그 이전 데이터는 디스크에 flush 함.

예를 들어 new SXSSFWorkbook(100)이면, 100개 row만 메모리에 유지, 나머진 임시 파일에 저장.

SXSSFWorkbook 작동 방식은 아래와 같다.

```sql
[ Row 0~99  ] => 메모리
[ Row 100~]  => 임시 디스크 파일로 flush

```

## 엑셀 파일 생성으로 보는 동기/논블로킹의 이점

동기/블로킹은 요청 하나당 스레드 1개를 점유하게 된다.(블로킹) 예를 들어서 5초 걸리는 요청이 동시에 100개면 스레드는 500초치 묶음이다. 서버가 일정 이상 트래픽을 못넘기고, 한 유저가 엑셀 요청중이면 다른 요청도 처리가 지연된다.(동기)

동기/논블로킹은
엑셀을 메모리에 올리지 않고, 조금식 만들어 스트리밍 전송하는 구조로 동작한다.

1. 스레드 자원 낭비를 방지: 스레드 논블로킹으로 서버 스레드가 막히지 않아 다른 요청도 병렬로 처리가 가능하다.
2. 메모리 정절약: SXSSF 같은 스트리밍 API는 row 수천개를 한꺼번에 안담는다.
3. 빠른 응답: 스트리밍 방식으로 인해 사용자 입장에서는 파일이 순차적으로 빨리 내려온다.
4. 대용량도 안전: 수십만 row 이상 생성해도 OOM 위험이 적다.

```java
@GetMapping("/excel")
public void generateExcel(HttpServletResponse response) throws IOException {
    response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
    response.setHeader("Content-Disposition", "attachment; filename=\"data.xlsx\"");

    try (SXSSFWorkbook workbook = new SXSSFWorkbook(100);  // 한 번에 100행만 메모리 유지
         OutputStream out = response.getOutputStream()) {

        Sheet sheet = workbook.createSheet("Sheet1");
        for (int i = 0; i < 100000; i++) {
            Row row = sheet.createRow(i);
            row.createCell(0).setCellValue("Row " + i);
        }

        workbook.write(out); // response에 바로 스트리밍 (non-blocking I/O와 호환)
    }
}
```

- SXSSWorbook: 스트리밍 방식(논블로킹 적합)
- write()는 실제로 버퍼 기반으로 I/O 처리 가능
- WebFlux에서는 Flux<DataBuffer> 형태로 엑셀 스트리밍도 완전 논블로킹 가능

  - 단, 엑셀 라이브러리가 대부분 블로킹 I/O라서 중간 단계에서 별도 워커로 분리 필요

## 동기/블로킹, 동기/논블로킹(Java NIO)

두 방식모두 결과가 필요해서 기다림(동기: 작업이 완료되기를 기다림)이라는 점에서 동기지만, 블로킹 방식은 기다리는 동안 스레드가 막혀있고, 논블로킹은 기다리기는 하지만, 스레드는 안막히고 다른 일이 가능하다.

| 유형                | 설명                                                                                                                                         |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **동기 + 블로킹**   | 식당에서 주문하고, 음식 나올 때까지 **테이블에서 기다림** (그 시간에 아무것도 못 함)                                                         |
| **동기 + 논블로킹** | 식당에서 주문하고, **호출기를 받고 테이블로 돌아가 다른 일 하다가** 벨 울리면 음식 받으러 감 (하지만 **내가 직접 처리하러 가야 함**, 동기적) |

스레드가 모든 데이터를 가지고 엑셆 파일 생성하는 요청 하나에 점유되느냐, 스레드가 스트리밍 방식으로 적은 데이터의 엑셀 생성하는 io작업동안 다른 엑셀파일도 생성하느냐?

---

### 참고자료

[[우아한 기술블로그] 아 엑셀다운로드 개발,,, 쉽고 빠르게 하고 싶다 (feat. 엑셀 다운로드 모듈 개발기)](https://techblog.woowahan.com/2698/)
기다리느냐 안기다리느냐 (비동기 논 블로킹)
집밖을 못나가면 비동기 블로킹
막혀있나 아니냐

# 스레드 풀 개수 조정 근거

핵심은 CPU의 노는 시간을 최대한 줄이도록 쓰레드 개수를 조정하면 된다.

## 쓰레드 풀의 적정 크기

> CPU 코어수 \* (1/cpu사용시간)

스레드 풀로 처리하고 싶은 요청의 전체 처리 시간 대비 CPU를 사용하는 시간의 역수를 코어 개수에 곱해주면 된다.

예를 들어서, CPU 코어 개수가 1개이고 어떤 요청을 처리할 때 CPU를 요청 처리 시간 중 1/3만 사용한다면 스레드 풀 크기는 최소 3은 되어야한다. 그래야 cpu가 작업을 완료했는데 스레드 풀 개수가 부족해서 요청을 못받으면 cpu는 일을 하지 않고 놀게 되는 리소스 낭비가 생긴다.

CPU 코어가 여러개이면 코어 개수 만큼 스레드 풀 크기를 늘려주면된다.

## 그럼 이 스레드 풀 크기로 1초당 처리할 수 있는 요청 수는?

> 리틀의 법칙, **L = λ W**
> 스레드 풀에서 사용중인 스레드 수 = 1초당 요청 수 \* 평균 요청 처리 시간

리틀의 법칙으로 1초당 처리할 수 있는 요청 수를 구할 수 있다.
예를 들어서, 스레드 풀의 크기가 50이고 평균 요청 처리 시간이 한 요청에 100ms라면 이 스레드 풀 크기로 우리가 1초당 처리할 수 있는 요청 수는 대략 50/0.1sec = 500 개 가 된다.
50 개의 스레드가 있고 이 스레드 들이 각각 요청 1개를 처리하는데 0.1 초가 걸린므로, 1초엔 500 개의 요청을 이 스레드 풀로 처리할 수 있다.

### 주의할 점은 데이터베이스의 커넥션 풀을 스레드 풀 내부의 요청들이 사용한다면 커넥션 풀의 크기를 꼭 확인해봐야한다.

![](../images/Pasted%20image%2020250718232017.png)
커넥션 풀을 많이 설정하였지만 DB 커넥션이 부족해서 DB 커넥션에서 병목현상이 발생할 수 있기 때문이다.

이렇게 데이터베이스 등의 외부 자원의 커넥션 풀을 스레드 풀 내부의 요청들에서 사용한다면 꼭 커넥션 풀 크기를 확인해봐야한다.
반대로 데이터베이스 커넥션 풀이 충분한데 스레드 풀이 작은 경우도, 자원을 제대로 활용하지 못하는 경우이다. 스레드 풀의 크기가 100인데 데이터베이스 커넥션 풀이 200이라면 데이터베이스 커넥션을 충분히 사용하지 못하는 경우이다.

또한 데이터베이스 커넥션 풀을 여러 스레드 풀이 함께 사용할 수 있다.

## 스레드 풀 크기를 정확히 계산하는 것보다, 부족하거나 과도하게 세팅하는 것을 막는게 중요

스레드 풀을 조정하기 위해서 매우 많은 요소를 세심하게 따져봐야한다.

- cpu 사용률
- 스레드풀을 사용하는 외부 자원
- 메모리 사용률(스레드가 많아지면 메모리 사용률도 늘어날 것이고, 여기서 메모리가 감당 가능한지도 살펴봐야한다.)
- 마치 서비스 벤치마크 테스트하는 것처럼...

cpu가 놀고 있거나 너무 과도하게 크게 설정하여 더 많은 자원을 불필요하게 사용하지 않게 만드는 것이 중요하다.

결국 스레드 개수 조정도 실험과학처럼 여러 시도를 해보고, 고쳐야한다고 생각한다.

## 이런 사고 방식을 어디에 활용할 수 있는가

스레드 풀 크기를 구하는 상황 이외에도 카프카 프로튜서와 컨슈머, 엘라스틱 서치 등의 세부 설정을 조절할 때 적절한 세팅값을 대략적으로 구하는 데 활용할 수 있다.
