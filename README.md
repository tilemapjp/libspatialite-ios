libspatialite-xamarin-mobile
=================

This repository is just under planning phase.
There are no schedule for completing works.

Fork元の libstatialite-iosのように、コロッとしたコンパイル結果をios、Android双方で作ってくれるスクリプトの生成を目的としています。

Xamarinにバインディングする事が目的ですが、とりあえずShared Objectの形にすれば、Objective-C/Javaのネイティブドライバへのバインディング=>c# ラッパのバインディングではなく、sqliteのsoを読んで叩いてくれる.NETの実装（まだ見つけてないが、Mono.Data.Sqliteを差し替えればできる？）がもしあるなら、それを差し替えていきなりso => XamarinのSpatialiteドライバが作れるかも。


目標
----

# iOS

* so化（iOSなのでdylib?よく判らん）する（現状、各ライブラリの静的ライブラリまで）。  
=>iOSでは共有ライブラリ使えないので、静的ライブラリまででよい模様。  
http://docs.xamarin.com/guides/ios/advanced_topics/native_interop/

# Android

* iOSを参考に、makeファイル作る  
=> 自分でtarball落としてきてso作るところまでやる  
=> 使うproj, geos, sqlite, spatialiteのバージョンもiOSに合わせる

* コンパイルの参考：  
https://code.google.com/p/spatialite-android/source/browse/spatialite-android-library/build.sh

* パッチ情報：  
https://code.google.com/p/spatialite-android/issues/detail?id=5  
https://code.google.com/p/spatialite-android/source/browse/#git%2Fspatialite-android-library%2Fjni%2Fpatches

* Android NDKでのコンパイル、及びコンパイル完了後のXamarinバインディング等はこちらの記事が参考になりそう  
http://takeshich.hatenablog.com/entry/2013/12/04/000000  
本家の標準ドキュメントはこちら  
http://docs.xamarin.com/guides/android/advanced_topics/using_native_libraries/  
http://docs.xamarin.com/guides/android/advanced_topics/cpu_architecture/
