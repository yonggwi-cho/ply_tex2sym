# ply_tex2sym

ply_tex2sym parses LaTeX math expressions and converts it into the equivalent SymPy form by PLY.  

Author:Akira Hakuta,  Date: 2017/04/23  

## Installation (windows)

TeX Live:  <http://www.tug.org/texlive/acquire-netinstall.html>

```
\texlive\2016\bin\win32\pythontex.exe
```

python3: <https://www.python.org/downloads/windows/>  
sympy : <http://www.sympy.org/en/index.html>  
ply : <http://www.dabeaz.com/ply/>  
```
pip install sympy
pip install ply
```


## Usage
```
python.exe tex2sym_parser.py
   --> Generating LALR tables
   
variable : a,b,...,z,\alpha,\beta,\gamma,\theta,\oomega
constant : pi --> \ppi, imaginary unit --> \ii, napier constant --> \ee

ply_tex2sym LaTeX expression style
\sin{x}
\cos^{2}{\theta}
\log{2}
\log{2}{8}
\frac{d}{dx}{(x^3+x^2+x+1)}
\int{(x^3+x^2+x+1) dx}
\int_{1}^{3}{(x-1)(x-3)^2 dx}
\lim_{x \to -\infty} {(\sqrt{x^2+3x}+x)}
\sum_{k=1}^{n}{k(k+1)^2}
\left| 3 - \ppi \right|
a_{n}


pdflatex.exe -synctex=1 -interaction=nonstopmode example.tex  
pythontex.exe example.tex  
pdflatex.exe -synctex=1 -interaction=nonstopmode example.tex  
```
## Examples

```
tex2sym(r'2^3') --> (2) ** (3)
tex2sym(r'2ab^2c^3') --> 2*a*(b) ** (2)*(c) ** (3)
tex2sym(r'\sqrt{3x}') --> sqrt(3*x)
tex2sym(r'\frac{2}{3}') --> (2) * (3)**(-1)
tex2sym(r'\sin {\ppi x}') --> sin(pi*x)
tex2sym(r'\log{\ee^3}') --> log((E) ** (3))
tex2sym(r'\log_{2}{8}') --> log(8)*(log(2)**(-1))
tex2sym(r'\frac{d}{dx}{x^3}') --> diff((x) ** (3),x)
tex2sym(r'\int{\sin^{2}{x} dx}') --> integrate((sin(x))**(2),x)
tex2sym(r'\sum_{k=1}^{n}{k^3}') --> summation((k) ** (3),(k,1,n))
tex2sym(r'\lim_{x \to -\infty}{(\sqrt{x^2+3x}+x)}') --> limit((sqrt((x) ** (2) + 3*x) + x), x, ((-1)*(oo)))
tex2sym(r'12a_{n+1}-35a_{n}') --> 12*F(n + 1) - 35*F(n)
tex2sym(r'2x^2+3x+4=0') --> Eq(2*(x) ** (2) + 3*x + 4,0)
tex2sym(r'x^2-3x-4 \leq 0') --> (x) ** (2) - 3*x - 4<=0
tex2sym(r'\left| \left| 3-\ppi \right|-1\right|') --> Abs(Abs(3 - pi) - 1)
```

### in japanese 2017/04/23版

## ply_tex2sym は LaTeX の数式コードを解析して、SymPy のコード変換する Python のプログラムツールです。
すでに、antlr4 で作られたLaTeX2SymPy <https://github.com/augustt198/latex2sympy> があります。
今回、python の構文解析ライブラリ PLY で作ってみました。



### 各ソフトのインスツール
#### TexLive
<https://www.tug.org/texlive/acquire-netinstall.html>  
install-tl-windows.exe でインスツールする。  
\texlive\2016\bin\win32の中にpythontex.exeがあります。  
これを使う！  

#### Python3
まず、<https://www.python.org/downloads/windows/> に入って、  
python3 の好きなバージョン、32bit、64bitを選び、インスツールして下さい。
コマンドプロンプトで  
pip install sympy  
と打ち込む。Successfully installed ...　と表示されればOK!  
\Python35\Lib\site-packagesのなかにパッケージのフォルダができる。  
ply も使うので、pip install ply

### 使い方
python tex2sym_parser.py
を実行すると、
```
Generating LALR tables
WARNING: 29 shift/reduce conflicts
WARNING: 76 reduce/reduce conflicts
WARNING: reduce/reduce conflict in state 58 resolved using rule (expr -> expr MINUS expr)
....
```

なるメッセージがでて、幾つかのファイルが生成されます。
WARNINGが出ますが、
... resolved using rule (expr -> expr MINUS expr)
2回目以降、WARNINGは出ません。
出力をみると
tex2sym_parser.tex2sym(texexpr)
でどんな変換ができるか、分かると思います。

更に
```
pdflatex.exe -synctex=1 -interaction=nonstopmode example.tex
pythontex.exe example.tex
pdflatex.exe -synctex=1 -interaction=nonstopmode example.tex
```
を実行すると、example.pdf が作成できます。

### 各ファイルの説明
**** tex2sym_lexer.py ****  
ply.lex.lex() は字句解析機を構築します。
これを用いて、文字列をtokenの並びへと変換します。
python.exe tex2sym_lexer.py
を実行して下さい。
どんな単語をtokenとして認識しているのかが分かります。
tokenの定義する方法は２つあります。
例えば、非負整数のtokenの名前を'NN_INTEGER'とします。
```
t_NN_INTEGER(t)= r'\d+'

def t_NN_INTEGER(t):
	r'\d+'
    return t
```
どちらでもtokenを定義できるのですが、
関数で定義すると、定義された順に高い優先度を与えられます。
例えば、
```
def t_NN_INTEGER(t):
    r'\d+'
    return t
    
def t_NN_FLOAT(t):
    r'\d*\.\d+'
    return t
```
の順序で定義すると、
'123.456'は
[('123', 'NN_INTEGER'), ('.456', 'NN_FLOAT')] のように２つのtokenと認識してしまいます。
定義の順序を入れ替えると
[('123.456', 'NN_FLOAT')]となります。
このように、順序に気を付けながらtokenを定義します。

**** tex2sym_parser.py ****  
ply.yacc.yacc() は 構文解析器を構築します。
これを用いて、定義されたルールに従って、LaTex の数式コードを SymPy のコード変換します。
```
# expr : expr^expr
def p_expr_exponent(p):
    'expr : expr EXPONENT expr'
    # p[0]  p[1]  p[2]    p[3]
    p[0] = '({}) ** ({})'.format(p[1], p[3])
```
で tex2sym(r'2^3') --> (2) ** (3) となります。
’expr : expr EXPONENT expr' は意味のある文字列で、
上記コメントのように、配列pの各要素とシンボル expr,EXPONENT の値が対応しています。
二重根号、繁分数式 等が正しく処理できるように、必要なp_ 関数を定義していきます。
高校数学レベルの数式を対象としました。

関数 mylatex(sympyexpr),mylatexstyle(texexpr) で
変数 : \alpha,\beta,\gamma,\theta,\oomega
定数 : pi --> \ppi, imaginary unit --> \ii, napier constant --> \ee
を使えるようにしています。

絶対値の記号 |expr| は曖昧な記号です。
次の式は2通りに解釈できます。
```
| 2|-3+4|-5 | = | 2-5 | = 3
| 2|-3+4|-5 | = 2 - 3 + 20 = 19
```
\left| expr \right| で定義をすることにします。
   

```

**** example.tex ****  
pythontex について  
\begin{pycode}  
code  
\end{pycode}  
codeの部分にpythonのコードを書き込みます。  

\pyc{code}はcodeを実行するコマンド。複数のコマンドを実行するのであれば、; を間に入れる。  
pyはpython、cはcommandの意味。

\py{value}は、valueを可能ならば文字列に変えて出力するコマンドのようです。  
\py{'text'}と\pyc{print('text')} は共に、文字列 text を出力します。  


tex2sym_parser.mylatex(sympyexpr), tex2sym_parser.mylatexstyle(texexpr)
で一部のギリシャ文字と &pi;, i, e が使える用に、置き換えをしています。

### モジュールのimport  
他のモジュールと同様に、  
from tex2sym_parser import tex2sym, mylatex, mylatexstyle
だけで import できるようにするには、  
まず、ダウンロードしたフォルダー ply_tex2sym-master を、Python35\Lib\site-packages にコピーまたは移動し,
Python35\Lib\site-packages に、例えば、  
ply_tex2sym-master  
の1行だけのファイル ply_tex2sym-master.pth を作ります。  
pythonは .pth の付いたファイルを読み込んで path を設定します。絶対path でもOK。  





