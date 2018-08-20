---
title: "2018 08 08 Ue4 Installed Buildをしてみた"
date: 2018-08-08T22:05:12+09:00
---

UE4の非配布版を使っているとリビルドをした時に問題がUE4までリビルドされたり  
UE4側も恐らく(間違っているかも)変更チェックとかをしていてビルドが非常に遅いなーと感じます  
EpicはUnrealEngine4を配布ビルドとしてビルドする方法を提供しているみたいなので試してみます。  

# 環境  

今回はUE4.20.1を使用します。  
ビルドにはVisualStudio2017を使用します。  
今回は解説を省きますのでhistoriaさんのブログを参照してください  

[historia:[UE4] エンジンのソースコード取得とビルド手順のまとめ UE4.6改訂版](http://historia.co.jp/archives/1327/)  
[UE4.20.1](https://github.com/EpicGames/UnrealEngine/tree/4.20.1-release)  



# やってみる

[UE4:InstalledBuild公式ドキュメント](http://api.unrealengine.com/JPN/Programming/Development/InstalledBuildReference/)  

公式ドキュメントを参考にUnrealAutomationToolを実行する  
ドキュメントをほぼ移しただけなのでそこまで詳しく把握はしていません  

ビルドする際には使わない＋ビルド出来ないターゲットプラットフォームを除外する必要があります  
実行時の引数に下記のように追記を加えると除外＋含める事ができます  
分散ビルドなどの高速にビルドする手段が無い場合は出来る限り使わないプラットフォームのビルドを避けるようにしましょう！  
```cmd
-set:WithWin64=true -set:WithAndroid=false
```

実際にビルドで使ったコマンド　　

```cmd:コマンドプロンプト
${UnrealEngineをgitで落としてきた場所}}\Engine\Build\BatchFiles\RunUAT.bat BuildGraph -target="Make Installed Build Win64" -script=Engine/Build/InstalledEngineBuild.xml -set:HostPlatformOnly=true -set:WithDDC=false -set:WithFullDebugInfo=true -clean
```

ビルドに成功すると下記の場所に配布用のビルドが作成されます  
${UnrealEngineをgitで落としてきた場所}}\LocalBuilds\Engine\Windows  

プロジェクトを作ってみると、画像のような構成になっていれば成功！！  
![ビルド構成のプルダウン](/images/20180820ue4installed_build.png)

# 失敗例  

ビルド成功するまで幾つか失敗をしたので対処法を載せます  

## PdbCopy.exeが無くて失敗する  

### UE4.19以前  

UE4.19はVisualStudio2015想定なのでパスが直書きされています。  
そのため下記を強制的に差し替えるようにしてください  

```cs
	public override void StripSymbols(FileReference SourceFile, FileReference TargetFile)
	{
		// 省略
		ProcessStartInfo StartInfo = new ProcessStartInfo();
		string PDBCopyPath = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.ProgramFilesX86), "MSBuild", "Microsoft", "VisualStudio", "v14.0", "AppxPackage", "PDBCopy.exe");
```

### PdbCopy.exe自体が存在しない

VisualStudio2017ではインストーラーを出来る限り軽量にする為に依存を限りなく少なくしようとする試みがされています。  
VS15.5からPdbCopy.exeが付属されないようになりました。  
フォーラムの方ではWindows10SDKをインストールをするようにと指示がされます  
[Developer CommunityのPdbCopy.exeの報告](https://developercommunity.visualstudio.com/content/problem/168414/pdbcopyexe-missing-from-155.html)  



![Win10SDKエラー](/images/20180820winsdk_error.png)  
すでにVisualStudioInstallerでインストールしている際には上記のようなエラー文が表示されます。  
VisualStudioInstallerで管理している物に手を加えるのが怖かったので**Debugging Tools for Windows**のみをオフラインインストールしました  

以下自己責任でおねがいします  

```cmd
winsdksetup.exe /layout
```
上記のコマンドを打つとインストーラーをダウンロードするディレクトリを聞かれるのでお好きな場所を指定してください。  
![Win10SDKエラー](/images/20180820winsdkoffline00.png)  

ダウンロードするインストーラーを聞かれるので**Debugging Tools for Windows**以外のチェックを外してください  

![Win10SDKエラー](/images/20180820winsdkoffline01.png)  

インストールを進めると画像のインストーラーが手に入るのでそれを使ってインストールをしてください。  
それでPdbCopy.exeがSDKにインストールされます。  

![Win10SDKエラー](/images/20180820winsdkoffline02.png)  
