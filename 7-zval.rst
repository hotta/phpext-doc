================
7.PHP の内部構造
================

7.1.zval
========

　PHP スクリプト（ユーザーランド）では、変数やオブジェクトは頭に ``$`` を付けるとか、関数やクラスはシンボルの前に ``function`` や ``class`` が来るなど、文法的にさまざまな取り決めがあります。

　一方、PHP の内部（Zend Engine）では、すべての変数やオブジェクトは、それぞれが zval と呼ばれるコンテナによって統合的に管理されています。Extension を開発するにあたっては、ほとんどの内部処理は zval やそのメンバーに対して操作を行なうことになりますので、zval の理解は必須となります。ここでは zval の概要をご紹介します。

　zval は、Zend/zend_types.h で定義されています。::

  typedef struct _zval_struct     zval;
  
  struct _zval_struct {
    zend_value        value;      /* value */
    union {
      struct {
        ZEND_ENDIAN_LOHI_4(
          zend_uchar    type,     /* active type */
          zend_uchar    type_flags,
          zend_uchar    const_flags,
          zend_uchar    reserved)     /* call info for EX(This) */
      } v;
      uint32_t type_info;
    } u1;
    union {
      uint32_t     next;                 /* hash collision chain */
      uint32_t     cache_slot;           /* literal cache slot */
      uint32_t     lineno;               /* line number (for ast nodes) */
      uint32_t     num_args;             /* arguments number for EX(This) */
      uint32_t     fe_pos;               /* foreach position */
      uint32_t     fe_iter_idx;          /* foreach iterator index */
      uint32_t     access_flags;         /* class constant access flags */
      uint32_t     property_guard;       /* single property guard */
      uint32_t     extra;                /* not further specified */
    } u2;
  };

　zval は、以下の３つのエリアに分けられます。

.. list-table::
  :widths: 5 10 10 50
  :header-rows: 1

  * - No.
    - 型
    - 変数名
    - 説明
  * - 1
    - zend_value
    - value
    - オブジェクトの値（一部例外あり）
  * - 2
    - union
    - u1
    - オブジェクトの型や属性
  * - 3
    - union
    - u2
    - Zend Engine が内部的に利用する


　Extension で意識しないといけないのは、そのオブジェクトの値を保持する zend_value と、その属性を示す u1 共用体の中のメンバーです。もっとも、これらのメンバーをソース中に記載することはまずありません。基本的に Zend ディレクトリ配下にある膨大なマクロを駆使して、これらのメンバーにアクセスすることになります。

　後半の u2 共用体は、コメントを見ると foreach が使っていそうとかなんとなく想像できるものもありますが、これらは基本的に Zend Engine 自身が使っているもので、Extension を開発する分には特に意識する必要はありません。

7.2.zend_value
==============

　zend_value の定義も Zend/zend_types.h の中にあります。::

  typedef union _zend_value {
    zend_long         lval;       /* long value */
    double            dval;       /* double value */
    zend_refcounted  *counted;
    zend_string      *str;
    zend_array       *arr;
    zend_object      *obj;
    zend_resource    *res;
    zend_reference   *ref;
    zend_ast_ref     *ast;
    zval             *zv;
    void             *ptr;
    zend_class_entry *ce;
    zend_function    *func;
    struct {
      uint32_t w1;
      uint32_t w2;
    } ww;
  } zend_value;

　これらは union ですので、利用の際はこれらのいずれか一つを選択することになります。オブジェクトの型と、それに対応するプロパティを以下に示しますします。

.. list-table::
  :header-rows: 1

  * - No.
    - 型
    - 変数名
    - PHPにおける型
    - アクセサ・マクロ（値渡し）
  * - 1
    - zend_long
    - lval
    - 整数(integer)
    - Z_LVAL(zval)
  * - 2
    - double
    - dval
    - 浮動小数点数(float/double)
    - Z_DVAL(zval)
  * - 3
    - zend_refcounted
    - \*counted
    - －
    - Z_COUNTED(zval)
  * - 4
    - zend_string
    - \*str
    - 文字列(string)
    - Z_STR(zval)
  * - 5
    - zend_array
    - \*arr
    - 配列(array)
    - Z_ARR(zval)
  * - 6
    - zend_object
    - \*obj
    - `オブジェクト <http://php.net/manual/ja/language.oop5.references.php>`_
    - Z_OBJ(zval)
  * - 7
    - zend_resource
    - \*res
    - `リソース <http://php.net/manual/ja/language.types.resource.php>`_
    - \Z_RES(zval)
  * - 8
    - zend_reference
    - \*ref
    - `リファレンス <http://php.net/manual/ja/language.references.whatare.php>`_
    - Z_REF(zval)
  * - 9
    - zend_ast_ref
    - \*ast
    - 抽象構文木
    - Z_AST(zval)
  * - 10
    - zval
    - \*zv
    - －
    - Z_INDIRECT(zval)
  * - 11
    - void
    - \*ptr
    - －
    - Z_PTR(zval)
  * - 12
    - zend_class_entry
    - \*ce
    - `クラス <http://php.net/manual/ja/language.oop5.php>`_
    - Z_CE(zval)
  * - 13
    - zend_function
    - \*func
    - `コールバック / Callable <http://php.net/manual/ja/language.types.callable.php>`_
    - Z_FUNC(zval)

　それぞれのプロパティに対して、対応するアクセサ・マクロが用意されています。たとえば保持している値が zend_long 値だとわかっている場合、値を取り出すためのアクセサは、値渡しなら ``Z_LVAL(zval)`` です。ポインタ渡しの場合のマクロ等のバリエーションについては後述します。

7.3.オブジェクトの型
====================

　（一部の例外を除いて）zval の値は zend_value に入っていますが、その値の型は (zval).u1 にある type / type_flags / type_info といったプロパティで管理されています。実際の型は、Zend/zend_types.h で以下のように定義されています。::

  #define IS_UNDEF          0
  #define IS_NULL           1
  #define IS_FALSE          2
  #define IS_TRUE           3
  #define IS_LONG           4
  #define IS_DOUBLE         5
  #define IS_STRING         6
  #define IS_ARRAY          7
  #define IS_OBJECT         8
  #define IS_RESOURCE       9
  #define IS_REFERENCE      10

　これらは必ずしも同じレベルではありません。たとえば bool 値を扱う場合 _IS_BOOL で判定しますが、実際にセットする値は IS_TRUE または IS_FALSE であり、セットする先も zend_value ではなく (zval).u1.type_info だったりします。ただマクロを使っている限りは、これらの差異を意識しなくても済みます。

　なお IS_STRING 以上の型については、それぞれに独自のコンストラクタとデストラクタを保つ場合があります。

7.4.アクセサ・マクロ
====================

　前項の型を判定するためのマクロや、値にアクセスするためのアクセサ・マクロには、値渡し以外にポインタ渡しバージョンもあります。 これらのマクロ定義は Zend/zend_types.h の中にあります。

.. list-table:: アクセサ・マクロ
  :header-rows: 1

  * - No.
    - 型名シンボル
    - | アクセサ（値／ポインタ）
    - | 判定（値／ポインタ）
  * - 1
    - IS_UNDEF
    - N/A
    - | Z_ISUNDEF(zval) /
      | Z_ISUNDEF_P(zval_p)
  * - 2
    - IS_NULL
    - N/A
    - | Z_ISNULL(zval)
      | ZVAL_IS_NULL(z) /
      | Z_ISNULL_P(zval_p)
  * - 3
    - | IS_FALSE
      | IS_TRUE
    - N/A
    - N/A
  * - 4
    - IS_LONG
    - | Z_LVAL(zval) / Z_LVAL_P(zval_p)
    - N/A
  * - 5
    - IS_DOUBLE
    - | Z_DVAL(zval) / Z_DVAL_P(zval_p)
    - N/A
  * - 6
    - IS_STRING
    - | Z_STR(zval) / Z_STR_P(zval_p)
    - （別途）
  * - 7
    - IS_ARRAY
    - | Z_ARR(zval) / Z_ARR_P(zval_p)
      | Z_ARRVAL(zval) / Z_ARRVAL_P(zval_p)
    - （別途）
  * - 8
    - IS_OBJECT
    - | Z_OBJ(zval) / Z_OBJ_P(zval_p)
      | Z_OBJ_HT(zval) / Z_OBJ_HT_P(zval_p)
      | Z_OBJ_HANDLER(zval, hf) 
      | / Z_OBJ_HANDLER_P(zv_p, hf)
      | Z_OBJ_HANDLE(zval) / Z_OBJ_HANDLE_P(zval_p)
      | Z_OBJCE(zval) / Z_OBJCE_P(zval_p)
      | Z_OBJPROP(zval) / Z_OBJPROP_P(zval_p)
      | Z_OBJDEBUG(zval,tmp) / Z_OBJDEBUG_P(zval_p,tmp)
    - （別途）
  * - 9
    - IS_RESOURCE
    - | Z_RES(zval) / Z_RES_P(zval_p)
      | Z_RES_HANDLE(zval) / Z_RES_HANDLE_P(zval_p)
      | Z_RES_TYPE(zval) / Z_RES_TYPE_P(zval_p)
      | Z_RES_VAL(zval) / Z_RES_VAL_P(zval_p)
    - （別途）
  * - 10
    - IS_REFERENCE
    - | Z_REF(zval) / Z_REF_P(zval_p)
      | Z_REFVAL(zval) / Z_REFVAL_P(zval_p)
    - | Z_ISREF(zval) / 
      | Z_ISREF_P(zval_p)




7.5.代入・返却用マクロ
======================

　代入用マクロには、直に値を代入する以外にも、値を初期化したりコピーしたりするものもあります。

　呼び出し元のユーザーランドに値を返には ``return_value`` （疑似）グローバル変数に値をセットする必要がありますが、この ``return_value`` も ``zval *`` 型です。 ``return_value`` に値をセットするだけのマクロ、および値をセットしてそのまま return する返却用マクロが用意されています。

.. list-table:: 代入・返却用マクロ
  :header-rows: 1

  * - No.
    - 代入
    - return_value への代入
    - 返却
  * - 1
    - ZVAL_UNDEF(z)
    - N/A
    - N/A
  * - 2
    - ZVAL_NULL(z)
    - RETVAL_NULL()
    - RETURN_NULL()
  * - 3
    - | ZVAL_BOOL(z, b)
      | ZVAL_FALSE(z)
      | ZVAL_TRUE(z)
    - | RETVAL_BOOL(b)
      | RETVAL_FALSE
      | RETVAL_TRUE
    - | RETURN_BOOL(b)
      | RETURN_FALSE
      | RETURN_TRUE
  * - 4
    - ZVAL_LONG(z, l)
    - RETVAL_LONG(l)
    - RETURN_LONG(l)
  * - 5
    - ZVAL_DOUBLE(z, d)
    - RETVAL_DOUBLE(d)
    - RETURN_DOUBLE(d)
  * - 6
    - | ZVAL_STR(z, s)
      | ZVAL_INTERNED_STR(z, s)
      | ZVAL_NEW_STR(z, s)
      | ZVAL_STR_COPY(z, s)
    - | RETVAL_STR(s)
      | RETVAL_INTERNED_STR(s)
      | RETVAL_NEW_STR(s)
      | RETVAL_STR_COPY(s)
      | RETVAL_STRING(s)
      | RETVAL_STRINGL(s, l)
      | RETVAL_EMPTY_STRING() 
    - | RETURN_STR(s)
      | RETURN_INTERNED_STR(s)
      | RETURN_NEW_STR(s)
      | RETURN_STR_COPY(s)
      | RETURN_STRING(s)
      | RETURN_STRINGL(s, l)
      | RETURN_EMPTY_STRING()
  * - 7
    - | ZVAL_ARR(z, a)
      | ZVAL_NEW_ARR(z)
      | ZVAL_NEW_PERSISTENT_ARR(z)
    - RETVAL_ARR(r)
    - RETURN_ARR(r)
  * - 8
    - ZVAL_OBJ(z, o)
    - RETVAL_OBJ(r)
    - RETURN_OBJ(r)
  * - 9
    - | ZVAL_RES(z, r)
      | ZVAL_NEW_RES(z, h, p, t)
      | ZVAL_NEW_PERSISTENT_RES
      |   (z, h, p, t)
    - RETVAL_RES(r)
    - RETURN_RES(r)
  * - 10
    - | ZVAL_REF(z, r)
      | ZVAL_NEW_EMPTY_REF(z)
      | ZVAL_NEW_REF(z, r)
      | ZVAL_NEW_PERSISTENT_REF(z, r)
      | ZVAL_UNREF(z)
      | ZVAL_COPY_UNREF(z, v)
    - N/A
    - N/A

7.6.特殊な型
============

　内部で使用する特殊な型もあります。これらは関数の戻り値にはなりません。

型の定義::

  /* constant expressions */
  #define IS_CONSTANT       11
  #define IS_CONSTANT_AST   12
  
  /* fake types */
  #define _IS_BOOL          13
  #define IS_CALLABLE       14
  #define IS_ITERABLE       19
  #define IS_VOID           18
  
  /* internal types */
  #define IS_INDIRECT       15
  #define IS_PTR            17
  #define _IS_ERROR         20
 
　これらに関するマクロには以下のようなものがあります。

.. list-table:: 特殊な型に関するマクロ
  :header-rows: 1

  * - No.
    - 型の定義
    - 判定用マクロ
    - アクセサマクロ
    - 代入用マクロ
  * - 1
    - | IS_CONSTANT
      | IS_CONSTANT_AST
    - | Z_CONSTANT(zval)
      | Z_CONSTANT_P(zval_p)
    - N/A
    - | Z_AST(zval) / 
      | Z_AST_P(zval_p)
      | Z_ASTVAL(zval) / 
      | Z_ASTVAL_P(zval_p)
      | ZVAL_NEW_AST(z, a)
  * - 2
    - _IS_ERROR
    - | Z_ISERROR(zval) / 
      | Z_ISERROR_P(zval_p)
    - N/A
    - ZVAL_ERROR(z)
  * - 3
    - IS_INDIRECT
    - N/A
    - | Z_INDIRECT(zval) /
      | Z_INDIRECT_P(zval_p)
    - ZVAL_INDIRECT(z, v)

　上記には記載しきれないマクロもまだ多数ありますので、これらのヘッダファイルをじ
っくりと研究する必要があります。なお、文字列や配列等については後述します。
