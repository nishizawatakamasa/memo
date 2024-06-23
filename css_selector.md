# CSSセレクタ覚書

## 単一の成分
* 基本セレクタ
    * \* (全称)
    * a (要素型)
    * .class (クラス)
    * #id (ID)
* 属性セレクタ
    * [attr] (attr属性を持つ)
    * [attr="value"] (attr属性の値が正確にvalueと一致)
    * [attr~="value"] (空白区切りの内の1つが正確にvalueと一致)
    * [attr|="value"] (正確にvalueと一致するか、 valueで始まり直後にハイフンが続く)
    * [attr^="value"] (valueで始まる)
    * [attr$="value"] (valueで終わる)
    * [attr*="value"] (文字列中にvalueを1つ以上含む)
    * [attr operator value i] (大文字と小文字を区別しなくなる。incensitive(鈍感)のi。)
* 疑似クラスセレクタ
    * ツリー構造疑似クラス
        * :empty
        * :nth-of-type()
        * :nth-last-of-type()
    * 関数疑似クラス
        * :is()
        * :not()
        * :has()

## 組み合わせ
* 密接(and)
* [半角スペース] 子
* \> 直下の子
* ~ 弟
* \+ 直後の弟

## 注意点
* 密接時、全称と要素型はクラス、ID、擬似クラスの前に置く。
* :has()を入れ子にはできない。
* 基本的に関数擬似クラスと疑似要素は相性悪い。
