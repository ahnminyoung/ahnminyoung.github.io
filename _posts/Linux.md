---

layout: post
title: "리눅스 명령어"

---

#리눅스

exit : 나가기
pwd : 경로확인

ll , ls : 폴더안의 파일확인
mkdir : 폴더 생성
history : 히스토리 까보자

cd    : 경로 이동

--------------------------------------------------------------------------------------------------------------------------------------------------------
vi    : 파일 생성및 편집
  -> i : 편집
  -> o : 다음라인에서 insert 편집
  -> esc : 명령취소
  -> dd : 삭제
  -> x : 한글자 지우기
  -> (숫자)gg : 라인넘버로 이동
  -> ]] : 맨밑으로 이동
  -> /(찾을단어) : 찾을단어로 이동
    -> n : 아래로 찾기
    -> 위로찾기 까먹음
  *-> :q : 나가기
  *-> :wq : 저장후 나가기
  *-> :q! : 강제로 나가기
  *-> :wq! : 강제로 저장하고 나가기

cat : 읽기모드로 파일열기
tail : 파일의 최하단 부분 표시
tail -f : 실시간 로그 보기
--------------------------------------------------------------------------------------------------------------------------------------------------------

cp (파일명) (경로)/(복사할 파일명) : 해당위치로 파일 복사
mv (파일명) (경로)/(변경할 파일명) : 해당위치로 파일 이동



chown : 소유자 변경관리
  ex) chown root test2     소유자 바꾸기
  ex) chwon root:wheel test2 그룹까지 바꾸기

chmod : 권환 변경 관리
  -> 4 : read
  -> 2 : write
  -> 1 : execute
  -> 0 : 아무권한도 없는것을 의미
  -> 7 : read, write, execute(4+2+1)권한들 조합
  -> 6 : read, write(4+2+0)권한들 조합
  -> 5 : read+execute (4+1)

  -> 첫번째 rwx 는 user

  ex:) chmod 775 test2              (755)제일많이쓴다

--------------------------------------------------------------------------------------------------------------------------------------------------------
ps : 프로세스 확인
  -> -e : 모든 프로세스를 출력
  -> -f : 풀 포멧으로 보여줌(UID, PID등)
  -> -l : 긴 포맷으로 보여줌

돌고있는 프로세스 확인할 때
  ps -ef | grep (java)
    -> java가 포함된 프로세스목록 표출
--------------------------------------------------------------------------------------------------------------------------------------------------------

grep : 원하는 문자열이 들어간 행을 찾아 출력하는 명령어

echo : 인수로 전달되는 텍스트/문자열을 표시하는데 사용



--------------------------------------------------------------------------------------------------------------------------------------------------------

모니터링

top

  -> 숫자 1 : CPU Core별로 사용량 확인

-CPU            (mpstat : cpu 찾는 다른방법)
  us : 유저레벨에서 사용하고 있는 CPU 비중
  sy : 시스템 레벨에서 사용하고 있는 CPU 비중
  **id : 유휴 상태의 CPU 비중 (CPU 사용량 : 100 - id)

  -메모리 (리눅스 메모리사용량)
  KiB Mem : 32683924 total,  3670772 free, 10259424 used, 18753728 buff/cache

  free : 메모리 사용량 확인
    -> -g : 기가확인
    -> -m : 메가확인


  disk용량

  df, df -P   : 디스크용량확인
    -> 사용중인 용량 : df -P | grep -v ^Filesystem | awk '{sum += $3} END { print sum/1024/1024 " GB" }'
    -> 총 용량     : df -P | grep -v ^Filesystem | awk '{sum += $2} END { print sum/1024/1024 " GB" }'
    -> 남은 용량    : df -P | grep -v ^Filesystem | awk '{sum += $4} END { print sum/1024/1024 " GB" }'
    -> 사용율      : df -P | grep -v ^Filesystem | awk '{sum += $5} END { print sum/100 " %" }'

sh 안에서
  #!/bin/sh : 이걸하고 작성함 먼지 나중에찾기



crontab (배치)특정시간에 특정 작업을 실행할 수 있음
  -> -e : 편집
  -> -l : 읽기
  -> -d : 삭제    쓰면안돼


로그 남길때
>> 경로/파일명 2>&1

--------------------------------------------------------------------------------------------------------------------------------------------------------

vi cpucheck.sh (cpu 사용율 체크 쉘)
  #!/bin/sh
  cpuAmount=`top -bn 1 | grep 0i cpu\(s\) | awk '{print 100 - $8"%"}'`

  date >> /home/twin/monitor/monitor.log 2>&1
  ehco "CPU : $cpuAmount" >> /home/twin/monitor/monitor.log 2>&1

vi diskcheck.sh (disk 용량 체크 쉘)
  #!/bin/sh
  disk=`df -P | grep -v ^Filesystem | awk '{sum += $3} END { print sum/1024/1024 " GB" }'`
  totalDisk=`df -P | grep -v ^Filesystem | awk '{sum += $2} END { print sum/1024/1024 " GB" }'`
  usedDiskRatio=`df -P | grep -v ^Filesystem | awk '{sum += $5} END { print sum/100 " %" }'`

  date >> /home/twin/monitor/monitor.log 2>&1
  echo "총용량 :$totalDisk" >> /home/twin/monitor/monitor.log 2>&1
  echo "사용중인용량 :$disk" >> /home/twin/monitor/monitor.log 2>&1
  echo "사용율 :$usedDiskRatio" >> /home/twin/monitor/monitor.log 2>&1

vi memorycheck.sh (메모리 용량 체크 쉘)
  #!/bin/sh
  totalMem=`free | grep ^Mem | awk '{print $2}'`
  availableMem=`free | grep ^Mem | awk '{print $7}'`

  realMemUsed=$((totalMem - availableMem))
  realMemUsed2=$((realMemUsed * 100 / totalMem))

  date >> /home/twin/monitor/monitor.log 2>&1
  echo "사용메모리: $realMemUsed2%" >> /home/twin/monitor/monitor.log 2>&1

sh cpucheck.sh , sh diskcheck.sh , sh memorycheck.sh (실행)

--------------------------------------------------------------------------------------------------------------------------------------------------------

sts 서버가 갑자기 듀플리케이트될때

lsof -i tcp:8090

kill -9 (PID)




cd /var/spool/mail/

vi root
cat root
