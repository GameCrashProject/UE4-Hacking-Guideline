# UE4-Hacking-Guideline

> [BoB(Best of the Best)](https://www.kitribob.kr/) 프로젝트의 일환으로 게임 엔진에 대한 취약점 분석 방법론에 대해 개발한다.
> 다양한 게임 엔진이 존재하나 다양한 이유로 우리는 상용 게임 엔진 중 Epic Games사의 Unreal Engine 4를 주 타겟으로 취약점 분석 방법론을 제시한다.

현재 게임 업계는 직접적인 운영과 관련된 매크로나 게임핵(Game Hack)과 같은 보안 위협을 방어하는데 중점을 두고 있으며 직접적인 운영과 관련되지 않은, 취약점에 대해서는 크게 관심을 두고 있지 않다. 마찬가지로 게임의 기반이 되는 게임 엔진에 대한 보안 위협 또한 다른 소프트웨어에 비해 관심이 낮다. 그래서 우리는 다양한 게임 엔진 중 대표적인 게임엔진인 Unreal Engine을 가지고 게임 엔진에 대한 취약점 분석을 진행한다.

4개월간 진행된 프로젝트로 후속 연구를 위한 가이드 라인을 제시하고자 한다.

# Author
> Best of the Best 10기 취약점분석 트랙 GameCrashProject(이하 GCP) 팀
- 심영진 멘토<sup id="head1">[M](#foot1)</sup>, 박천성 멘토<sup id="head1">[M](#foot1)</sup>
- 이도현 PL<sup id="head2">[9](#foot2)</sup>
- 김영민 PM<sup id="head3">[10](#foot3)</sup>, 구성민<sup id="head3">[10](#foot3)</sup>, 김훈민<sup id="head3">[10](#foot3)</sup>, 정다훈<sup id="head3">[10](#foot3)</sup>, 정원홍<sup id="head3">[10](#foot3)</sup>, 황시준<sup id="head3">[10](#foot3)</sup>

# Contents
## 서론
[1. UnrealEngine](https://github.com/GameCrashProject/UE4-Hacking-Guideline/blob/main/1_Introduction/1_UnrealEngine.md)<br>
[2. Attack Vectors](https://github.com/GameCrashProject/UE4-Hacking-Guideline/blob/main/1_Introduction/2_Attack_Vectors.md)<br>
[3. Tools](https://github.com/GameCrashProject/UE4-Hacking-Guideline/blob/main/1_Introduction/3_Tools.md)<br>

## 프로젝트 접근 방법론
[1. Build](https://github.com/GameCrashProject/UE4-Hacking-Guideline/blob/main/2_Methodology/1_Build.md)<br>
[2. Analysis](https://github.com/GameCrashProject/UE4-Hacking-Guideline/blob/main/2_Methodology/2_Analysis.md)<br>
[3. Exploit](https://github.com/GameCrashProject/UE4-Hacking-Guideline/blob/main/2_Methodology/3_Exploit.md)<br>
                                                                                                          
## 결론
[1. Conclusion](https://github.com/GameCrashProject/UE4-Hacking-Guideline/blob/main/3_Conclusion/1_Conclusion.md)<br>

---
<b id="foot1">[[M](#head1)]</b> KITRI 차세대 보안 리더 양성 프로그램 취약점 분석 트랙 멘토 <br>
<b id="foot2">[[9](#head2)]</b> KITRI 차세대 보안 리더 양성 프로그램 취약점 분석 트랙 9기 수료생<br>
<b id="foot3">[[10](#head3)]</b> KITRI 차세대 보안 리더 양성 프로그램 취약점 분석 트랙 10기 교육생<br>
