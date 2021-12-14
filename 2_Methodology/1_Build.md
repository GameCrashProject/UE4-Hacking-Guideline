### Contents <!-- omit in toc -->
- [1. Unreal Engine 빌드](#1-unreal-engine-빌드)
	- [1.1 소스코드 다운로드](#11-소스코드-다운로드)
	- [1.1 Windows 빌드](#11-windows-빌드)
	- [1.2. Linux 빌드](#12-linux-빌드)
	- [1.2. 빌드 시스템 분석](#12-빌드-시스템-분석)
- [2. AFL++ 빌드](#2-afl-빌드)
	- [2.1 전체 모듈 계측삽입](#21-전체-모듈-계측삽입)
		- [2.1.1 UnrealBuildTool 수정](#211-unrealbuildtool-수정)
		- [2.1.2 빌드 환경 설정](#212-빌드-환경-설정)
		- [2.1.3 빌드를 위한 소스코드 수정](#213-빌드를-위한-소스코드-수정)
		- [2.1.4 빌드](#214-빌드)
	- [2.2 부분 계측 삽입](#22-부분-계측-삽입)
		- [2.2.1 리스폰스 파일 수정 및 컴파일](#221-리스폰스-파일-수정-및-컴파일)
		- [2.2.2 링킹](#222-링킹)
	- [2.3 검증](#23-검증)
	- [3. WinAFL 빌드](#3-winafl-빌드)

---

# 1. Unreal Engine 빌드
언리얼 엔진의 소스코드를 다운로드 받기위해서는 깃허브 계정과 에픽 게임즈 계정을 연동해야된다. 상세한 방법은 다음 문서를 참조하라.

[계정연동](https://www.unrealengine.com/en-US/ue4-on-github)

## 1.1 소스코드 다운로드
계정 연동을 끝맞췄다면 git을 통해 소스코드를 다운로드 받을 수 있다. 해당 깃허브 저장소이기 때문에 미리 토큰을 발급받거나 SSH 공개키를 깃허브에 등록 하는 것이 좋다.

토큰으로 소스코드를 clone하고자 한다면 다음의 명령어를 실행하면 된다.

```sh
git clone https://github.com/EpicGames/UnrealEngine.git
```

SSH 공개키를 깃허브에 등록하였다면 다음의 명령어로 저장소를 clone할 수 있다.

```sh
git clone git@github.com:EpicGames/UnrealEngine.git
```

기본적으로 `release` 브랜치로 다운로드 되며 해당 브랜치는 최신 언리얼 엔진 버진이다. 에픽 게임즈측에서는 버전별로 태그를 달아놓았으며 구 버전을 다운로드 받아야 될 경우 `git checkout`으로 소스코드의 버전을 변경할 수 있다. 예를 들어서 4.26 버전으로 바꾸기 위해서는 `git checkout 4.26.0-release`를 실행함으로 다운그레이드 할 수 있다.

## 1.1 Windows 빌드
Windows의 경우 빌드를 하기 위해서는 Visual Studio 2017이상이 설치되어 있어야 된다. 상세한 빌드 방법에 대해서는 다음 레퍼런스를 참조하라.

[Unreal Engine 빌드하기](https://docs.unrealengine.com/4.27/ko/ProductionPipelines/DevelopmentSetup/BuildingUnrealEngine/)


## 1.2. Linux 빌드
Linux의 경우 먼저 다음과 같은 명령어로 프로젝트를 초기화 시켜야 된다.

```sh
$ ./Setup.sh
$ ./GenerateProjectFiles.sh
```
모든 작업이 완료되었다면 `make`를 통해 빌드를 할 수 있으며 에디터의 경우 `make UE4Editor`로 빌드를 할 수 있으며 게임의 경우 `make UE4Game`으로 빌드를 할 수 있다.

기본적으로 모든 빌드 구성은 `Development`이며 릴리즈 버전의 경우 `make UE4Game-Linux-Shipping`으로 빌드할 수 있다. 

## 1.2. 빌드 시스템 분석
UE4에서는 UnrealBuildTool(언리얼 빌드 툴, 이하 UBT)를 사용하여 빌드가 진행된다. 빌드할때 사용하는 컴파일러와 링커는 플랫폼마다 다르며 상세한 정보는 `Engine/Source/Programs/UnrealBuildTool/Platform`에서 각 소스코드를 분석해야된다. 해당 문서에서는 Linux에서의 빌드 과정을 다룬다.

`make`를 실행하게되면 `make`에서는 UBT를 실행한다. 해당 UBT에서는 정의된 정보에 따라서 각 모듈의 종속성과 상태를 결정하며 빌드할 모듈의 범위를 정한다. 그리고 나서 UnrealHeaderTool(UHT)를 사용하여 각 모듈에서 사용될 헤더 파일(.h)을 생성한다.

헤더 파일이 생성되면 컴파일을 위한 리스폰스 파일(.rsp)과 링킹과 심볼 분리를 위한 쉘 스크립트를 생성하며 컴파일러와 쉘 스크립트를 실행하면서 빌드가 진행된다.

# 2. AFL++ 빌드
퍼즈 테스트를 효율적으로 하기 위해서는 계측을 삽입해야 된다. 계측을 삽입하기 위해서는 각 퍼저에서 커스텀한 컴파일러를 사용해야된다. 우리는 전체 모듈에 대한 계측을 삽입하는 방법과 특정 모듈에만 계측을 삽입하는 방법에 대해서 연구를 하였다.

## 2.1 전체 모듈 계측삽입
### 2.1.1 UnrealBuildTool 수정
UnrealBuildTool(이하 UBT)에서 AFL++에서 사용하는 컴파일러인 `afl-clang-lto++`를 사용하도록 수정해야 된다. 해당 유틸리티의 소스코드는 Github의 저장소에서 `Source/Programs/UnrealBuildTool`에서 확인할 수 있다.

해당 유틸리티는 C#으로 작성이 되었으며 빌드는 Microsoft Visual Studio에서 해야된다. 해당 프로젝트에서는 `Platform/Linux` 디렉토리에 Linux 빌드와 관련된 소스코드를 확인할 수 있다.

`LinuxCommon.cs+47`의 소스코드를 다음과 같이 수정한다.

```diff
				return Path.Combine(InternalSDKPath, "bin", "clang++");
			}

-			string[] ClangNames = { "clang++", "clang++-7.0", "clang++-6.0" };
+			string[] ClangNames = { "afl-clang++-lto", "clang++", "clang++-7.0", "clang++-6.0" };
			string ClangPath;
```

`afl-clang-lto++`의 경우 `-x` 옵션을 지원하지 않는다. 하지만 UBT에서 기본적으로 해당 옵션을 삽입하며 해당 옵션을 사용하지 않도록 수정을 해야된다. 해당 옵션을 삽입하는 함수는 `LinuxToolChain.cs`에서 확인할 수 있으며 C++같은 경우 `GetCompileArguments_CPP`함수에서 해당 옵션을 삽입하며 C언어 같은 경우 `GetCompileArguments_C`함수에서 해당 옵션을 삽입한다.

두 함수의 소스코드를 다음과 같이 수정한다.

```diff
		static string GetCompileArguments_CPP(CppCompileEnvironment CompileEnvironment)
		{
			string Result = "";
-			Result += " -x c++";
			Result += GetCompilerStandardVersion_CPP(CompileEnvironment);
			return Result;
		}

		static string GetCompileArguments_C()
		{
			string Result = "";
-			Result += " -x c";
			return Result;
		}
``` 

또한 사용하는 툴체인을 변경해줘야 된다. 해당 부분 같은 경우 `LinuxToolChain.cs+119`부터 다음과 같이 수정을 하면 된다.

```diff
			if (bForceUseSystemCompiler)
			{
				// Validate the system toolchain.
				BaseLinuxPath = "";
				MultiArchRoot = "";

				ToolchainInfo = "system toolchain";

				// use native linux toolchain
				ClangPath = LinuxCommon.WhichClang();
				GCCPath = LinuxCommon.WhichGcc();
				ArPath = LinuxCommon.Which("ar");
				LlvmArPath = LinuxCommon.Which("llvm-ar");
-				RanlibPath = LinuxCommon.Which("ranlib");
-				StripPath = LinuxCommon.Which("strip");
-				ObjcopyPath = LinuxCommon.Which("objcopy");
+				RanlibPath = LinuxCommon.Which("llvm-ranlib);
+				StripPath = LinuxCommon.Which("llvm-strip);
+				ObjcopyPath = LinuxCommon.Which("llvm-objcopy);

				// if clang is available, zero out gcc (@todo: support runtime switching?)
				if (!String.IsNullOrEmpty(ClangPath))
				{
					GCCPath = null;
				}
```

AFL++의 경우 별도의 ld를 사용하므로 해당 ld를 사용하도록 변경을 해줘야 된다. `LinuxToolChain.cs+119`를 다음과 같이 수정하면 된다.

```diff
protected virtual string GetLinkArguments(LinkEnvironment LinkEnvironment)
		{
			string Result = "";

			if (UsingLld(LinkEnvironment.Architecture) && (!LinkEnvironment.bIsBuildingDLL || (CompilerVersionMajor >= 9)))
			{
-				Result += (BuildHostPlatform.Current.Platform == UnrealTargetPlatform.Win64) ? " -fuse-ld=lld.exe" : " -fuse-ld=lld";
+				Result += (BuildHostPlatform.Current.Platform == UnrealTargetPlatform.Win64) ? " -fuse-ld=lld.exe" : " -fuse-ld=-fuse-ld=afl-ld-lto";
			}
```

기본적으로 UBT에서는 내장된 표준 라이브러리를 가지고 빌드를 하나 계측을 하기 위해서는 LLVM의 라이브러리를 사용해야 된다. `LinuxToolChain.cs+561`과 `LinuxToolChain.cs+562`를 다음과 같이 수정한다.

```diff
			if (ShouldUseLibcxx(CompileEnvironment.Architecture))
			{
				Result += " -nostdinc++";
-				Result += " -I" + "ThirdParty/Linux/LibCxx/include/";
-				Result += " -I" + "ThirdParty/Linux/LibCxx/include/c++/v1";
+				Result += " -I" + "/usr/lib/llvm-11/include/c++/";
+				Result += " -I" + "/usr/lib/llvm-11/include/c++/v1";
			}
```

또한 `LinuxToolChain.cs+1911`부터 다음과 같이 수정한다.

```diff
			if (ShouldUseLibcxx(LinkEnvironment.Architecture))
			{
				// libc++ and its abi lib
				LinkCommandString += " -nodefaultlibs";
-				LinkCommandString += " -L" + "ThirdParty/Linux/LibCxx/lib/Linux/" + LinkEnvironment.Architecture + "/";
-				LinkCommandString += " " + "ThirdParty/Linux/LibCxx/lib/Linux/" + LinkEnvironment.Architecture + "/libc++.a";
-				LinkCommandString += " " + "ThirdParty/Linux/LibCxx/lib/Linux/" + LinkEnvironment.Architecture + "/libc++abi.a";
+				auto LLVMLibPath = "/usr/lib/llvm-11/lib/";
+				LinkCommandString += " -L" + LLVMLibPath;
+				LinkCommandString += " " + LLVMLibPath + "/libc++.so";
+				LinkCommandString += " " + LLVMLibPath + "/libc++abi.so"; 
				LinkCommandString += " -lm";
				LinkCommandString += " -lc";
				LinkCommandString += " -lpthread"; // pthread_mutex_trylock is missing from libc stubs
				LinkCommandString += " -lgcc_s";
				LinkCommandString += " -lgcc";
			}
```

정상적으로 계측을 하기 위해서는 AFL++ 전용 라이브러리가 추가적으로 포함되어야 된다. `LinuxtoolChain.cs+1705`에 다음과 같은 코드를 추가하면 된다.

```diff
				ResponseLines.Add(string.Format("\"{0}\"", InputFile.AbsolutePath.Replace("\\", "/")));
				LinkAction.PrerequisiteItems.Add(InputFile);
			}
+		ResponseLines.Add("/usr/local/lib/afl/afl-compiler-rt.o");
+		ResponesLines.Add(""/usr/local/lib/afl/afl-llvm-rt-lto.o");
```

UE4Game을 대상으로 하는 경우 엔진 기능에 대한 공유 라이브러리를 사용하지 않기 때문에 상관이 없지만 UE4Editor나 다른 유틸리티의 경우 공유 라이브러리를 사용하기 때문에 링크 스크립트를 수정해줘야 된다. AFL++를 확인하면 공유 라이브러리에 대한 계측을 하는 방법에 대해서 나와 있으며 해당 방법을 토대로 링크 스크립트를 수정한다.

간단히 공유 라이브러리에 계측을 하는 방법에 대해서 설명을 하자면 공유 라이브러리를 링킹할 때는 `AFL_LLVM_LTO_DONTWRITEID`가 1로 설정 되어야 되며 `AFL_LLVM_LTO_STARTID`를 0에서부터 증가 시켜줘야 된다. `AFL_LLVM_LTO_STARTID`는 다음 공유 라이브러리를 빌드하기 전에 수치를 증가시켜 줘야 되며 AFL++에서 링킹 결과를 기반으로 증가시켜야 된다. 마지막으로 실행파일을 빌드할때는 `AFL_LLVM_LTO_DONTWRITEID`가 설정되어 있으면 안된다.

수정할 코드의 위치는 `LinuxToolChain.cs+1978` 부터이며 다음과 같이 수정을 하였다.
```diff
				else
				{
					LinkWriter.NewLine = "\n";
					LinkWriter.WriteLine("#!/bin/sh");
					LinkWriter.WriteLine("# Automatically generated by UnrealBuildTool");
					LinkWriter.WriteLine("# *DO NOT EDIT*");
					LinkWriter.WriteLine();
					LinkWriter.WriteLine("set -o errexit");

+					if (LinkEnvironment.bIsBuildingDLL)
+						{
+							LinkWriter.WriteLine("AFL_LLVM_LTO_DONTWRITEID=1");
+							LinkWriter.WriteLine("AFL_LLVM_LTO_STARTID=`cat /tmp/AFL_LLVM_LTO_STARTID`");
+							LinkWriter.WriteLine("if [ -z $AFL_LLVM_LTO_STARTID ]; then");
+							LinkWriter.WriteLine("touch /tmp/AFL_LLVM_LTO_STARTID");
+							LinkWriter.WriteLine("\tAFL_LLVM_LTO_STARTID=0");
+							LinkWriter.WriteLine("fi");
+						}
+						else
+						{
+							LinkWriter.WriteLine("unset AFL_LLVM_LTO_DONTWRITEID");
+							LinkWriter.WriteLine("AFL_LLVM_LTO_STARTID=`cat /tmp/AFL_LLVM_LTO_STARTID`");
+						}
+						LinkWriter.WriteLine();
+						LinkWriter.WriteLine("echo \"Linking Start! - AFL_LLBM_LTO_STARTID:\" $AFL_LLVM_LTO_STARTID")
+						LinkWriter.Write("temp=`");
+						LinkWriter.Write(LinkCommandString);
+						LinkWriter.WriteLine("`");
+						if (LinkEnvironment.bIsBuildingDLL)
+						{
+							LinkWriter.WriteLine("temp=`echo $temp | sed 's/.*Instrumented //' | sed 's/ locations.*//' | grep --color=never -o '[0-9]\\+'`");
+							LinkWriter.WriteLine("AFL_LLVM_LTO_STARTID=$((AFL_LLVM_LTO_STARTID+temp))");
+							LinkWriter.WriteLine("echo $AFL_LLVM_LTO_STARTID > /tmp/AFL_LLVM_LTO_STARTID");
+						}
+						else 
+						{
+							LinkWriter.WriteLine("echo 0 > /tmp/AFL_LLVM_LTO_STARTID");
+							LinkWriter.WriteLine("echo \"Linking End! - AFL_LLBM_LTO_STARTID:\" `cat /tmp/AFL_LLVM_LTO_STARTID`");
+						}
+					LinkWriter.WriteLine(GetDumpEncodeDebugCommand(LinkEnvironment, OutputFile));
+				}
```

그냥 실행시킬 경우 동기화 문제가 발생한다. 동기화를 방지하기 위해서 리눅스의 기본 유틸리티 중 하나인 `flock`을 사용하도록 코드를 수정한다. `LocalExecutor.cs+70`의 `ThreadFunc`를 수정하면 된다.

```diff
			ActionStartInfo.RedirectStandardOutput = false;
			ActionStartInfo.RedirectStandardError = false;

			// Log command-line used to execute task if debug info printing is enabled.
			Log.TraceVerbose("Executing: {0} {1}", ActionStartInfo.FileName, ActionStartInfo.Arguments);
+			if (processStartInfo.FileName == "/bin/sh")
+			{
+				ActionStartInfo.UseShellExecute = true;
+				ActionStartInfo.FileName = "/usr/bin/flock";
+				ActionStartInfo.Arguments = "/tmp/AFL_LLVM_LTO_STARTID -c \"sh " + ActionStartInfo.Arguments.Replace('"', ' ') + "\"";
+			} 
```

### 2.1.2 빌드 환경 설정
먼저 아래의 명령어를 통해 수정한 UnrealBuildTool에서 사용하는 파일을 생성한다.
```shell
echo 0 > /tmp/AFL_LLVM_LTO_STARTID
```

그리고 아래와 같이 Build.sh 파일을 수정해준다. 다음 명령어를 통해 수정한 UBT와 시스템에 설치된 컴파일러를 가지고 빌드할 수 있다.

```shell
sed -i '13,18s/^/#/' Engine/Build/BatchFiles/Linux/Build.sh
sed -i '21s/ / -ForceUseSystemCompiler /2' Engine/Build/BatchFiles/Linux/Build.sh
```

AFL++에서 커스텀한 컴파일러의 경우 PCH를 제대로 지원하지 않으며 UHT에 계측이 삽입되는 경우 오류가 발생하니 다음과 같이 `BuildConfiguation.xml`를 수정해야 된다.
```shell
echo -e "<?xml version=\"1.0\" encoding=\"utf-8\" ?>\n" \
	"<Configuration xmlns=\"https://www.unrealengine.com/BuildConfiguration\">\n" \
	"\t<BuildConfiguration>\n" \
	"\t\t<bUsePCHFiles>false</bUsePCHFiles>\n" \
	"\t</BuildConfiguration>\n" \
	"\t<UEBuildConfiguration>\n" \
	"\t\t<bDoNotBuildUHT>true</bDoNotBuildUHT>\n" \
	"\t<\<UEBuildConfiguration>\n" \
	"</Configuration>" | sed 's/^ //' > Engine/Saved/UnrealBuildTool/BuildConfiguration.xml
```

UBT는 `Engine/Binaries/DotNET`에 위치해 있으며 해당 UBT를 수정해서 빌드한 UBT로 변경한다.

### 2.1.3 빌드를 위한 소스코드 수정
PCH가 활성화 되어있을 경우 오류가 발생하지 않으나 PCH가 비활성화 되어 있을 경우 컴파일 에러가 발생하며 소스코드를 수정해야 된다.

`Engine/Plugins/Developer/RiderSourceCodeAccess/Source/RiderSourceCodeAccess/Private/RiderPathLocator/Linux/RiderPathLocatorLinux.cpp`를 다음과 같이 수정해야된다.

먼저 12번째에 변경할 함수를 사용할 수 있게 헤더파일을 추가해줘야 된다.
```diff
#if PLATFORM_LINUX
+ #include "Unix/UnixPlatformProcess.h"
```

그리고 나서 79번째 라인의 `FPlatfomProcess`를 `FUnixPlatfromProcess`로 수정하면 된다.

```diff
- FPlatfomProcess::ExecProcess(TEXT("/usr/bin/mdfind"), TEXT("\"kMDItemKind == Application\""), &ReturnCode, &OutResults, &OutErrors);
+ FUnixPlatformProcess::ExecProcess(TEXT("/usr/bin/mdfind"), TEXT("\"kMDItemKind == Application\""), &ReturnCode, &OutResults, &OutErrors);
```

`Engine/Plugins/FX/Niagara/Source/NiagaraEditor/Public/Widgets/SItemSelector.h`의 경우 다음 헤더파일을 추가하면 된다.

```c
#include "Framework/Application/SlateApplication.h"
```

`Engine/Source/Runtime/Core/Public/Templates/TypeHash.h`에는 다음 코드를 상단에 추가하면 된다.

```c
#include <stdint.h>
```

`Engine/Plugins/Media/ImgMedia/Source/ImgMedia/Private/Assets/ImgMediaMipMapInfo.cpp`와 `Engine/Source/Runtime/MovieScene/Private/MovieSceneCommonHelpers.cpp`에는 다음 코드를 상단에 추가하면 된다. 
```c
#include "Engine/Engine.h"
```

### 2.1.4 빌드
`make`를 통해 빌드를 하게 되면 빌드가 된다. 하지만 링킹하는 과정에서 64기가 이상의 메모리를 요구하며 속도도 매우 느린 문제가 발생하였기에 특정 모듈에 계측을 할 수 있는 방법을 연구하였다.

## 2.2 부분 계측 삽입
해당 방법의 경우 UBT 수정 없이 빌드가 가능하며 리스폰스 파일과 쉘 스크립트를 빌드하고 나서 지우지 않는 것을 이용한 방법이다. 

만약 전쳬계측을 위해 패치가 되어 있다면 UBT와 `Engine/Build/BatchFiles/Linux/Build.sh`를 원상태로 복구해야 된다. 간단히 `Engine/Build/BatchFiles/Linux/Build.sh`의 주석을 친 부분을 제거하면 최초 실행시 UBT를 기본 UBT로 변경한다. `Engine/Saved/UnrealBuildTool/BuildConfiguration.xml`의 경우 를 전체 계측과 동일하게 수정한다.

이후 UBT와 UE4Game을 이용하여 빌드한다.
```shell
make UnrealBuildTool 
make UE4Game
```

이제 개별 소스코드에 대해서 AFL++를 적용한다.

### 2.2.1 리스폰스 파일 수정 및 컴파일
`Engine/Intermediate/Build/Linux/B4D820EA/UE4/Development`에서 AFL++를 적용할 모듈을 선정하고 리스폰스 파일을 수정한다. 해당 예제에서는 `PakFile`을 사용하였다.

```sh
$ cd Engine/Intermediate/Build/Linux/B4D820EA/UE4/Development/PakFile
$ sed -i 's/-x c++//' Module.PakFile.cpp.o.rsp
$ export RSP_PATH="`pwd`/Module.PakFile.cpp.o.rsp"
$ cd Engine/Source
$ afl-clang-lto++ @"$RSP_PATH"
```

### 2.2.2 링킹
컴파일이 정상적으로 되었다면 다음과 같은 명령어로 쉘 스크립트 파일을 수정하고 링킹을 하면 계측을 삽입할 수 있다.

```shell
$ export UNREALENGINE_LIB_PATH="$(pwd)/Engine/Source/ThirdParty"
$ export UNREALENGINE_LIB_PATH=$(echo $UNREALENGINE_LIB_PATH | sed -e 's/\//\\\//g')
$ cd Engine/Intermediate/Build/Linux/B4D820EA/UE4/Development
$ sed -i 's/.*clang++"/afl-clang-lto++/' Link-UE4Game-ASan.link.sh
$ sed -i 's/-LThirdParty/-L'"$UNREALENGINE_LIB_PATH"'/' Link-UE4Game-ASan.link.sh
$ sed -i 's/ ThirdParty/ '"$UNREALENGINE_LIB_PATH"'/' Link-UE4Game-ASan.link.sh
$ sed -i 's/ ThirdParty/ '"$UNREALENGINE_LIB_PATH"'/' Link-UE4Game-ASan.link.sh
$ sh Link-UE4Game-ASan.link.sh
```

## 2.3 검증
AFL++의 계측이 정상적으로 삽입되었는지 확인하기 위해 `readelf`와 `gdb`로 확인을 할 수 있다.

```shell
$ cd Engine/Binaries/Linux
$ readelf -Ws UE4Game-ASan | grep afl # 결과에 afl 관련 함수나 오브젝트가 보이면 적용이 된것.
$ gdb UE4Game-ASan
(gdb) disassemble 함수명 # 디스어셈블링된 코드 중에
						# 0x1dca682(%rip),%rax # 0x18f934a0 <__afl_area_ptr> <- 이런식으로 나오면 적용이 된것.
```

리눅스에서 게임을 실행시키기 위해서는 빌드한 UE4Game을 패키지 디렉토리에 옮기고 실행시키면 된다.


## 3. WinAFL 빌드
WinAFL은 Windows에서 데이케이트 서버에 대한 퍼즈 테스트를 위해 사용한 퍼저로 Visual Stduio 버전과 DynamoRio 위치는 각 환경에 맞추어주면 된다.

```batch
> git clone --recurse-submodules https://github.com/googleprojectzero/winafl.git
> cd winafl
> mkdir build64
> cd build64
> cmake -G "Visual Studio 16 2019" -A x64 .. -DDynamoRIO_DIR="C:\Users\kunsh\Desktop\DynamoRIO\cmake"
> cmake --build . --config Release
```

아래의 명령어로 퍼저를 실행시킨다.
```batch
> .\afl-fuzz.exe -debug -i "D:\Fuzzing\results\input\" -o "D:\Fuzzing\results\output\" -D "D:\Fuzzing\DynamoRIO-Windows-8.0.0-1\bin64" -t 5000 -- -covtype edge -coverage_module "D:\Unreal\Package\WindowsServer\Dedicated\Binaries\Win64\DedicatedServer.exe" -target_module "D:\Unreal\Package\WindowsServer\Dedicated\Binaries\Win64\DedicatedServer.exe" -target_offset 0x2632b50 -persistence_mode in_app -- "D:\Unreal\Package\WindowsServer\Dedicated\Binaries\Win64\DedicatedServer.exe"
```