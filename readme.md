[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.2249217.svg)](https://doi.org/10.5281/zenodo.2249217)

 LIDCデータ処理スクリプト
 ============================
 
※本ドキュメントは[原文](https://github.com/MIC-DKFZ/LIDC-IDRI-processing/readme.md)を日本語訳したものです。

このリポジトリに含まれるスクリプトはLIDC-IDRIデータの変換に利用できます。このスクリプトをコールすれば、画像および領域分割データがnifti/nrrd形式で、結節の所見はシングルカンマ区切り（CSV）形式で利用できるようになります。

論文でこのスクリプトを利用した場合には、下記の引用を記載願います。

Michael Goetz, "MIC-DKFZ/LIDC-IDRI-processing: Release 1.0.1", DOI: 10.5281/zenodo.2249217


## 要件
このスクリプトは、いくつかの標準Pythonライブラリ（glob, os, subprocess, numpy, xml）とPythonライブラリSimpleITKを使用しています。また、MITKのコマンドラインツールを使用しています。
これらの使用のため、MITKのビルドおよび分類モジュールの有効化で入手するか、必要なすべてのコマンドラインツールを含んだ[MITK Phenotyping](http://mitk.org/Phenotyping)のインストールをしてください。

## 基本的な使用方法
 * [LIDC-IDRI](https://wiki.cancerimagingarchive.net/display/Public/LIDC-IDRI) のウェブサイトからデータをダウンロードしてください。画像データとしてDICOMファイルと説明データとしてXMLファイル（放射線科医のアノテーションおよび領域分割（XML形式）データ）が必要になります。NBIAデータ検索を利用してデータをダウンロードする際、デフォルトの"Descriptive Directory Name"オプションではなく、"Classic Directory Name"オプションを選択してください。
 * [MITK Phenotyping](http://mitk.org/Phenotyping)をビルドするか、ダウンロードおよびインストールをしてください。
 * "lidc_data_to_nifti.py"のファイル中に記載されたパスを登録してください。
 * "lidc_data_to_nifti.py"のスクリプトを実行してください。
 
下記の通り、入力パスを定義する必要があります：
 * path_to_executables : MITK Phenotypingのコマンドラインツールへのパス
 * path_to_dicoms : DICOM画像ファイル（領域分割のDICOMファイルではなく）が含まれるフォルダ
 * path_to_xmls : 結節について記載されたXMLを含むフォルダ

下記の通り、出力パスを定義する必要があります：
 * path_to_nrrds : NrrdもしくはNiftiファイルの出力先のフォルダ
 * path_to_planars :各検査の平面画像の出力先フォルダ
 * path_to_characteristics : CSVファイルへのパス。結節の所見が記憶されます。すでにファイルが存在する場合には、追記されます。
 * path_to_error_file : エラーメッセージが出力されるエラーファイルへのパス。すでにファイルが存在する場合には、追記されます。

## 出力結果

本スクリプトで生成される出力結果は、DICOMシリーズ全体（例：完全な３次元CT画像データ）を含んだNrrdファイルと、結節の3次元領域分割データのNifti (.nii.gz)ファイルと、結節の領域分割されたスライスを含んだNrrdおよび平面画像（.pf）から構成されます。

これらデータは<患者ID>ごとに分けてサブフォルダに記憶されます。<患者ID>の5文字はLIDC_IDRI Dicomフォルダ内で利用される患者IDの数字部分と合致しています。ただし、何名かの患者は１つ以上のCT画像を含む場合があるため、<患者ID>に1文字追記されます。したがって、各CTスキャンはユニークな<患者ID>となっています。例えば、フォルダ"LIDC_IDRI-0129" は、２つのCT画像を含んでいる場合、<患者ID>は"0129a"と"0129b"のようになります。

各患者と画像に対して最大４つの読影セッションがあります。
<セッションID>は、与えられた画像に対する専門家の範囲を示す1文字の数字になります。論文によれば、各セッションは12名の専門家のうち1名によって行われています。しかし、同じ専門家がアノテーションした２つの画像が必ず確保されている可能性はありません。
したがって、２つの画像は場合によっては、同じ＜セッションID＞であっても、異なる専門家によってアノテーションされている可能性があります。

結節と専門家の各組み合わせは、例えば0000358のように8文字のユニークな数字＜結節ID＞で表されます。
このIDは全ての結節の領域分割および専門家の組み合わせに対してユニークとなっています。
すなわち、同じ結節の２つの領域分割は異なる＜結節ID＞を持つことを意味します。
これに対し、8文字の数字で表された＜正結節ID＞は同じ結節に対する全ての領域分割で同一となっています。
これは、与えられた結節の全ての領域分割の＜結節ID＞の最小値となるように定義されています。

＜ROI ID＞はある一つの結節の2次元領域分割か正面画像の中でユニークなIDとなっています。これは、同じ対象物の領域分割にて異なる複数の断面を区別するために使われます。

これら定義に基づき、下記のようなファイルが生成されます:
 * `path_to_nrrds/<Patient ID>/<Patient_ID>_ct_scan.nrrd` : 3DCT画像を含むnrrdファイル
 * `path_to_nrrds/<Patient ID>/<Patient_ID>_<Session ID>_<Nodule ID>_<True Nodule ID>.nii.gz` : 結節の領域分割を含んだNiftiファイル
 * `path_to_nrrds/<Patient ID>/planar_masks/<Patient_ID>_<Session ID>_<Nodule ID>_<ROI ID>.nrrd `: 結節の領域分割のある一断面を含んだNrrdファイル
 * `path_to_planars/<Patient ID>/<Patient_ID>_<Session ID>_<Nodule ID>_<ROI ID>.pf` : 結節の領域分割のある一断面を含んだ正面画像ファイル

補足として、結節の所見は出力パスpath_to_characteristicsで指定されたファイル内に記録されます。同様に全体処理の中でエラーが起きた場合は、同様にpath_to_error_fileで指定されたファイルへ書き出されます。
 
## 注意事項
このスクリプトはWindows環境で開発されています。Linux環境でも実行できる可能性がありますが、一切テストされていません。サブプロセス（例えば、MITKフェのタイピングの実行処理のコールなど）が呼ばれたことで問題が引き起こされる可能性があります。

また、このスクリプトは著者自身の研究のために開発されており、広範囲にテストはされません。いくつかの制限が誤って含まれている可能性があります。

LIDC_IDRIがオンラインで提供するDICOM領域分割ファイルがなかった頃にこのスクリプトを開発しました。従って、このスクリプトはXML記述に依存しています。これは、最適な解決手段とはいえないかもしれません。現在のDICOM領域分割オブジェクトを扱える新しい解決手段について、お気軽に拡張もしくはコード記述してください。

## 問い合わせ先
もし、質問や提案などありましたら、著者 (Michael Goetz) のメールアドレスm.goetz@dkfz-heidelberg.de
からコンタクトできます。

## ライセンス（原文ママ）

Copyright (c) 2003-2019 German Cancer Research Center,
Division of Medical Image Computing
All rights reserved.

Redistribution and use in source and binary forms, with or
without modification, are permitted provided that the
following conditions are met:

 * Redistributions of source code must retain the above
   copyright notice, this list of conditions and the
   following disclaimer.

 * Redistributions in binary form must reproduce the above
   copyright notice, this list of conditions and the
   following disclaimer in the documentation and/or other
   materials provided with the distribution.

 * Neither the name of the German Cancer Research Center,
   nor the names of its contributors may be used to endorse
   or promote products derived from this software without
   specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND
CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES,
INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR
CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE
GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR
BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT
OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.
