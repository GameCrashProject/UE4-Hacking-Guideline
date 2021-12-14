### Contents <!-- omit in toc -->
- [1. 개요](#1-개요)
- [2. Fuzzing](#2-fuzzing)
  - [2.1 AFL++](#21-afl)
  - [2.2 peach](#22-peach)
  - [2.3 WinAFL](#23-winafl)
- [3. Analyzing](#3-analyzing)
  - [3.1 Static Code Analysis Tools](#31-static-code-analysis-tools)
  - [3.2 Debugging Tools](#32-debugging-tools)
    - [3.2 Dynamic Analysis Tools](#32-dynamic-analysis-tools)
  
---

# 1. 개요
UE4 취약점 분석 시에 사용한 도구를 정리하였다.

# 2. Fuzzing

## 2.1 AFL++
리소스 파일 퍼징을 위해 사용한 도구로, 코드 커버리지 기반 퍼저 중 널리 알려진 퍼저이며 화이트 박스 퍼징과 블랙 박스 퍼징을 모두 지원하기에 사용한 도구다. 하지만 화이트 박스 퍼징을 위하여 계측 삽입을 위한 빌드가 필요하며 언리얼 엔진의 빌드 시스템을 분석하여 계측을 삽입한 후 퍼징을 진행하였다.

## 2.2 peach
오디오 파일 퍼징을 위해 선정한 스마트 퍼저로 공개된 Wav의 구조를 기반으로 Wav 파일을 뮤테이션하여 에디터에서 임포트 하는 방식으로 퍼징을 진행하였다.

## 2.3 WinAFL
네트워크 퍼징을 위해 네트워크와 관련한 다양한 기능을 제공하는 WinAFL을 사용하였다. 서버와 클라이언트의 인증을 위한 시간 딜레이를 주는 기능도 가능하며 추가적으로 빌드가 필요하다. 하지만 현재 타임아웃과 관련한 오류로 제대로 동작하지 않는 문제가 발생하였다.

# 3. Analyzing 
## 3.1 Static Code Analysis Tools
UE4는 오픈 소스이기에 소스 코드에 대한 직접적인 분석이 가능하나 코드 베이스만 350MB 이상으로 소스코드만 읽는 방식으로 분석이 불가능하기 때문에 IDE와 서드파티 코드 분석 플러그인을 사용하여 정적 코드 분석을 진행하였다.

- Microsoft Visual Studio
- JetBrains ReSharper C++

## 3.2 Debugging Tools
정적 코드 분석만으로는 전체적인 구조나 취약점 분석하기가 쉽지 않기 떄문에 디버거를 사용하여 동작 방식과 취약점 분석을 진행하였으며, 퍼징 결과를 분석할때도 사용하였다.

- Microsoft Visual Studio 내장 디버거
- Microsoft Windbg
- GDB (Pwndbg)

### 3.2 Dynamic Analysis Tools
네트워크 패킷이나 함수의 인자를 분석하기 위해 사용한 도구로 사용하였다. 

- Wireshark
- frida