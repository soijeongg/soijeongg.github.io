---
layout: post
read_time: true
show_date: true
title: "프로메테우스를 적용해보자 !"
date: 2024-04-08 15:00:20 -0600
description: "프로메테우스를 적용해보자 !"
image: https://example.com/path/to/image.png
tags: 
    - coding
    - diary
    - hh99
author: soi

toc: no # leave empty or erase for no TOC
---
왜 Jmeter는 프로그램인데 왜 프로메테우스나 그라파나는 로컬 호스트로 들어가야 하는건지 모르겠지만 

### 의사결정 
 프로메테우스의 기본적인 포트는 9090
그라파나의 기본 포트는 3000 
그래서 그라파나의 기본 포트를 바꿔 줘야 한다 지금 안 바꾸면 둘이 겹쳐서 그라파나가 실행이 안되더라 
나는 맥의 brew로 설치했다 
**포트 바꾸는 방법**
내가 너무 고생해서 적는 방법

1. 그라파나를 homebrew로 설치한다
2. brew info grafana를 통해 설치 위치를 확인한다 
3.  --config /opt/homebrew/etc/grafana/grafana.ini  이 부분을 본다 
4. nano /opt/homebrew/etc/grafana/grafana.ini 
nano 편집기를 사용해 설정파일을 연다 
5. 아래로 내리다가 http_port 부분을 찾는다 아마 ;http_port = 3000 이렇게 되어있을텐데 이걸 원하는 포트 번호로 바꾼다 
**설정파일의 앞의 ;은 주석이기 때문에 이걸 지워야 한다**
6. 변경사항 저장을 위해 Ctrl + X를 누르고 Y를 누르고 Enter
7. 서버를 재시작 brew services restart grafana
![](https://velog.velcdn.com/images/soijeongg/post/58e52dd2-51fa-40db-90da-a2a9b12762b9/image.png)
3000번에서 3001번으로 바꾸고 연결완료!
### 설치 
1. brew에 입력
brew reinstall prometheus
brew services start prometheus
![](https://velog.velcdn.com/images/soijeongg/post/ac15031c-cd71-4a6a-af48-61f97a8c8947/image.png)
우리 파일을 적용하기 위해 맥에서  nano /opt/homebrew/etc/prometheus.yml
그리고 여기가서 했는데 오류나서 나는 그냥 그 파일에 들어가서 vscode로 열기로 했다 
- 맥에서 비밀 파일을 열기 위해 커맨드 shift .을 눌러 핀더에서 숨겨진 폴더 보기 
다운로드에서 상위폴더 -> 유저에서 또 상위폴더 
거기서 숨겨진 폴더에 opt가 있다 
거기 들어가면 homebrew존재 
![](https://velog.velcdn.com/images/soijeongg/post/263b73c0-d644-4245-b124-d65c22fc1b5e/image.png)
etc안의 prometheus.yml을 vscode로 열기 

### 그라파나와 프로메테우스 
프로메테우스와 그라파나는 모니터링을 위한 도구 
보통 프로메테우스로 상태데이터를 수집하고 그라파나로 사용자가 보기 쉽게 시각화한다 
모니터링 수집도구는 프로메테우스 말고도 데이터독, 인플럭스 DB등이 있지만 오픈 소스를 활용하는 기업은 프로메테우스외 다른 선택지가 없을 정도로 가장 탁월한 효율을 자랑함

데이터를 시각화하는 도구는 그라파나 말고도 키바나 크로노그래프등이 있으나 업계에서는 그라파나와 키바나가 시장을 양분화한 상태 
하지만 키바나는 프로메테우스와 연결 구성이 복잡해 프로메테우스 사용할때는 간결하게 사용할 수 있는 그라파나를 더 선호

### 프로메테우스 사용법 
**맥 버전 ** 
1. home brew를 사용해 프로메테우스를 설치한다 
2. brew info prometheus를 입력한다 
prometheus_brew_services` and uses the flags in:
   /opt/homebrew/etc/prometheus.args 이 부분을 주목
3. 여기를 들어간다 
cd /opt/homebrew/etc/
그후 ls prometheus* 를 입력해 파일을 찾는다 

4.nano /opt/homebrew/etc/prometheus.yml를 사용해 파일을  vs code로 연다 
![](https://velog.velcdn.com/images/soijeongg/post/38c8e603-1b94-4afd-9308-4a66fc9515bc/image.png)
입력하고 restart를 하면 드디어!!! localhost:9090이 열린다 
![](https://velog.velcdn.com/images/soijeongg/post/b869b4c7-48e4-4b83-ba96-c4ff3af4cbdc/image.png)
### express에 작용
express에 적용하기 위해 prom-client를 설치한다 (yarn add prom-client)
- 설치하는 이유는? 
프로메테우스는 프로젝트의 엔드포인트에 접속해서 검사를 하는데(위의 캡쳐를 보면 엔드포인트라고 되어있음)
그래서 엔드포인트를 만들어줘야 한다 
근데 아무거나 주면 될까? 이 서버전체를 검사하기 위한걸 줘야 한다 
```javascript
//metrics.js

import { Gauge } from "prom-client";

// 메모리 사용량을 측정하는 Gauge 메트릭 생성
const memoryUsageMetric = new Gauge({
  name: "node_memory_usage_bytes",
  help: "Total memory usage of the node process in bytes",
});

// 메트릭 값을 업데이트하는 함수
const updateMemoryUsageMetric = () => {
  const memoryUsageInBytes = process.memoryUsage().heapUsed;
  memoryUsageMetric.set(memoryUsageInBytes);
};

export { memoryUsageMetric, updateMemoryUsageMetric };
```
이렇게 prom-client 설정 파일을 따로 빼서 사용한다 
```javascript
//app.js
import { updateMemoryUsageMetric, memoryUsageMetric } from '../../metrics.js';
// 프로메테우스가 메트릭을 수집할 수 있는 엔드포인트 설정
app.get('/metrics', async (req, res) => {
  try {
    const metrics = await register.metrics();
    res.set('Content-Type', register.contentType);
    res.send(metrics);
  } catch (error) {
    console.error('Failed to generate metrics:', error);
    res.status(500).json({ error: 'Failed to generate metrics' });
  }
});
```
이렇게 적용한다 
이제 그래프를 어떻게 그리지..?

prom-client의 각각의 요소 정리 
1. Gauge(게이지)
단일 숫자 값을 가지며 언제든지 증가하거나 감소할 수 있는 메트릭
시스템의 상태나 순간적인 값을 표시하는데 사용
예를 들어, 메모리 사용량, CPU 사용량, 현재 연결된 사용자 수 등을 측정할 때 사용

2. Counter(카운터)
Counter는 단조 증가하는 값으로, 오직 증가할 수만 있는 메트릭
Counter는 주로 이벤트의 횟수나 요청 수와 같이 측정이 증가하는 값을 표시하는데 사용

3. Histogram(히스토그램)
Histogram은 주어진 샘플의 분포를 나타내는 메트릭으로, 값이 범위에 따라 분류
Histogram은 주로 이벤트의 분포나 지연 시간과 같이 범위에 따른 데이터 분포를 측정할 때 사용
 HTTP 요청의 응답 시간을 측정할 때 사용
 범위에 따라 샘플이 그룹화되므로, 각 범위에 해당하는 버킷(bucket) 값과 샘플 개수를 포함
 
![](https://velog.velcdn.com/images/soijeongg/post/59b6caab-6aed-41c3-819b-0ed5633618fb/image.png)
제대로 엔드포인트를 설정하면 이렇게 up이라고 초록색이 나오게 된다 

### 코드 
gpt에 물어보니 프로메테우스가 모니터링 해야할것
1. 시스템 리소스:  CPU 사용량, 메모리 사용량, 디스크 사용량, 네트워크 트래픽 등
2. 응용 프로그램 상태:요청 수, 응답 시간, 오류 발생률
3. 서버 상태 및 가용성: 서버의 가용성과 상태를 모니터링하여 장애를 감지하고 신속하게 대응

이중 어차피 node expoter로 넘어갈꺼서 기본적인걸 메트릭 수집히기로ㅓ 했다 
사실은 node expoter를 시스템 수준에서 사용할 수 있다고 해서 일단 할 수 있는걸 한게 맞지만
1. 노드별 메모리 사용률
2. 시스템 CPU 사용률 
3. 시스템 디스크 사용량 
4. HTTP 요청 총수
5. HTTP 요청 처리 시간
```javascript
import { Gauge } from "prom-client";
import os from "os";
import osUtils from "node-os-utils";

// 메모리 사용량을 측정하는 Gauge 메트릭 생성
const memoryUsageMetric = new Gauge({
  name: "node_memory_usage_bytes",
  help: "Total memory usage of the node process in bytes",
});

// CPU 사용량을 측정하는 Gauge 메트릭 생성
const cpuUsageMetric = new Gauge({
  name: "system_cpu_usage_percentage",
  help: "CPU usage percentage of the system",
});

// 디스크 사용량을 측정하는 메트릭 생성
const diskUsageMetric = new Gauge({
  name: "system_disk_usage_bytes",
  help: "Disk usage of the system in bytes",
});

// CPU 사용량을 가져오는 함수
const getCpuUsage = () => {
  const cpus = os.cpus();
  let totalIdle = 0;
  let totalTick = 0;

  cpus.forEach((cpu) => {
    for (const type in cpu.times) {
      totalTick += cpu.times[type];
    }
    totalIdle += cpu.times.idle;
  });

  return {
    idle: totalIdle / cpus.length,
    total: totalTick / cpus.length,
  };
};
//cpu를 업데이트 하는 함수
const updateCpuUsageMetric = async () => {
  try {
    const startMeasure = getCpuUsage();

    setTimeout(async () => {
      const endMeasure = await getCpuUsage();

      const idleDifference = endMeasure.idle - startMeasure.idle;
      const totalDifference = endMeasure.total - startMeasure.total;

      const percentageCpuUsage = 100 - (100 * idleDifference) / totalDifference;
      cpuUsageMetric.set(percentageCpuUsage);
    }, 100); // 측정 주기 설정
  } catch (error) {
    console.error("Failed to update CPU usage metric:", error);
  }
};

// 메모리 값을 업데이트하는 함수
const updateMemoryUsageMetric = () => {
  try {
    const memoryUsageInBytes = process.memoryUsage().heapUsed;
    memoryUsageMetric.set(memoryUsageInBytes);
  } catch (error) {
    console.error("Failed to update memory usage metric:", error);
  }
};
//디스크 사용률을 계산하는 함수
async function updateDiskUsage() {
  try {
    const diskInfo = await osUtils.drive.info();
    // 바이트 단위 대신 메가바이트 단위로 디스크 사용량을 계산
    const diskUsageInBytes =
      diskInfo.totalGb * 1024 * 1024 * 1024 * (diskInfo.usedPercentage / 100);
    diskUsageMetric.set(diskUsageInMB);
  } catch (error) {
    console.error("Error updating disk usage:", error);
  }
}
// 타이머를 사용하여 매 초마다 CPU 및 메모리 메트릭을 업데이트
setInterval(() => {
  updateCpuUsageMetric();
  updateMemoryUsageMetric();
  updateDiskUsage();
}, 1000); // 1초마다 업데이트

export {
  memoryUsageMetric,
  updateMemoryUsageMetric,
  cpuUsageMetric,
  updateCpuUsageMetric,
  diskUsageMetric,
  updateDiskUsage,
};
``` 
이렇게 설정 파일을 사용하고  app.js에 연동한다 
```javascript
const httpRequestCounter = new Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'path', 'status'],
});
register.registerMetric(httpRequestCounter);
// Express 미들웨어로 요청 처리 시 카운터 증가
// Express 미들웨어로 요청 처리 시 카운터 증가
// HTTP 요청 처리 시간 측정을 위한 히스토그램 생성
const httpRequestDurationHistogram = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'path', 'status'],
  buckets: [0.1, 0.5, 1, 2, 5], // 요청 처리 시간 범위
});
register.registerMetric(httpRequestDurationHistogram);

// Express 미들웨어로 요청 처리 시 카운터 증가 및 처리 시간 측정
app.use((req, res, next) => {
  const start = process.hrtime(); // 요청 시작 시간 기록
  // 현재 HTTP 요청에 대한 카운터 증가
  httpRequestCounter.inc({
    method: req.method,
    path: req.path,
    status: res.statusCode,
  });

  // 요청이 완료된 후 처리 시간 측정 및 히스토그램에 추가
  res.on('finish', () => {
    const durationInMilliseconds = getDurationInMilliseconds(start); // 요청 처리 시간 계산
    httpRequestDurationHistogram.observe(
      {
        method: req.method,
        path: req.path,
        status: res.statusCode,
      },
      durationInMilliseconds / 1000 // 초 단위로 변환
    );
  });

  next();
});

register.setDefaultLabels({ app: 'stocking' }); // 선택적으로 기본 레이블을 설정할 수 있습니다.
register.registerMetric(memoryUsageMetric);
register.registerMetric(cpuUsageMetric);
register.registerMetric(diskUsageMetric);
// /metrics 엔드포인트를 생성하여 Prometheus 서버로 메트릭을 노출합니다.
app.get('/metrics', async (req, res) => {
  try {
    res.set('Content-Type', register.contentType);
    const metrics = await register.metrics();
    res.send(metrics);
  } catch (error) {
    console.error(error);
    res.status(500).send('Internal Server Error');
  }
});

// CPU 및 메모리 메트릭을 업데이트하는 함수를 호출하여 주기적으로 업데이트합니다.
setInterval(() => {
  updateMemoryUsageMetric();
  updateCpuUsageMetric();
}, 1000);
``` 
app.js에 연동한다 
이러면 프로메테우스에서 확인 할 수 있게 된다 
![](https://velog.velcdn.com/images/soijeongg/post/0f2d5594-d0e0-4f50-9d0a-b934fc234fad/image.png)
이 엔드포인트를 클릭하면
![](https://velog.velcdn.com/images/soijeongg/post/d8b4b1f3-1ee9-44a5-8e91-9958626e05ce/image.png)

### 그라파나 연동
그라파나의 대시보드에서 찾아보니까 다른 사람들꺼 import해서 사용한다고 해서 했는데 다 no data떠서 처음에는 진짜 멘붕이였다 
근래서 찾아보다 대시보드에서 적용할 수 있는 방법을 찾았다 
https://www.inflearn.com/questions/267420/%EB%8C%80%EC%89%AC%EB%B3%B4%EB%93%9C%EC%97%90-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%A5%BC-%EB%AA%BB%EA%B0%80%EC%A0%B8%EC%98%A4%EB%8A%94%EA%B2%83-%EA%B0%99%EC%8A%B5%EB%8B%88%EB%8B%A4
이 사이트가 인프런에서 나온건데 여기 보니까 쿼리가 있더라
그래서 이 쿼리를 찾기 위해 설정파일을 찾고 있었는데 데이터소스 프로메테우스 explore거 있더라 여기 들어가서 쿼리가 처음에는 빌더인데 코드로 바꾸니까 나오더라 
그래서 각각 지표를 넣으니까 그래프가 나왔다 
그리고 대시보드에 저장하고 대시보드에 들어갔더니 aadd query가 있어서 하나씩 추가할 수 있었다 
![](https://velog.velcdn.com/images/soijeongg/post/4b2ef690-ff63-4171-8593-9be05ea73d89/image.png)
그래프 연동완료!

다음에는 node expoter를 사용해서 전체 모니터링 하는걸 글로 써봐야겠다 