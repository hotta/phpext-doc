===============
11.文字列の扱い
===============

11.1.文字列の内部構造
=====================

　Zend Engine における文字列は、zend_string として保持されます。zend_string は Zend/zend_types.h により以下のように実装されています。::

  typedef struct _zend_string     zend_string;
  
  struct _zend_string {
    zend_refcounted_h gc;
    zend_ulong        h;                /* hash value */
    size_t            len;
    char              val[1];
  };
  
  typedef struct _zend_refcounted_h {
    uint32_t         refcount;      /* reference counter 32-bit */
    union {
      struct {
        ZEND_ENDIAN_LOHI_3(
          zend_uchar    type,
          zend_uchar    flags,    /* used for strings & objects */
          uint16_t      gc_info)  /* keeps GC root number (or 0) and color */
      } v;
      uint32_t type_info;
    } u;
  } zend_refcounted_h;

11.2.文字列関連マクロ
=====================

　文字列に関する基本的なマクロは `7.PHP の内部構造 </phpext/html/7-zval.html>`_ でご紹介した通りですが、細かい操作のマクロやインライン関数は Zend/zend_string.h で定義されています。

.. list-table:: 文字列関連マクロ - ショートカット
  :header-rows: 1

  * - No.
    - マクロ宣言
    - 定義
    - 意味
  * - 1
    - ZSTR_VAL(zstr)
    - (zstr)->val
    - 文字列値
  * - 2
    - ZSTR_LEN(zstr)
    - (zstr)->len
    - 文字列の長さ
  * - 3
    - ZSTR_H(zstr)
    - (zstr)->h
    - ハッシュ値
  * - 4
    - ZSTR_HASH(zstr)
    - zend_string_hash_val(zstr)
    - （後述）

　ハッシュ値は、その文字列を一意に識別するための整数値です。これにより、文字列を高速に検索できるようになっています。


11.3.インライン関数
===================

　多数のインライン関数が用意されています。関数宣言において、冗長な部分は記載を省略しています。

.. list-table:: 文字列関連のインライン関数
  :header-rows: 1

  * - No.
    - 関数宣言
    - 意味
  * - 1
    - | zend_ulong 
      | zend_string_hash_val(zend_string \*s)
    - | ・ハッシュ値が未計算なら計算・保存する
      | ・ハッシュ値を返す
  * - 2
    - | void 
      | zend_string_forget_hash_val(zend_string \*s)
    - ハッシュ値をクリアする
  * - 3
    - | uint32_t
      | zend_string_refcount(const zend_string \*s)
    - 文字列のリファレンスカウントを返す(\*1)
  * - 4
    - | uint32_t
      | zend_string_addref(zend_string \*s)
    - リファレンスカウントをインクリメントして返す(\*1)
  * - 5
    - | uint32_t
      | zend_string_delref(zend_string \*s)
    - リファレンスカウントをデクリメントして返す(\*1)
  * - 6
    - | zend_string
      | \*zend_string_alloc(size_t len, int persistent)
    - 文字列用の領域を確保して返す
  * - 7
    - | zend_string 
      | \*zend_string_safe_alloc
      | (size_t n, size_t m, size_t l, int persistent)
    - 文字列用の領域を確保して返す（詳細調査中）
 
* (\*1)…いずれも拘束文字列（'interned string'）の場合は 1 を返します。拘束文字列として定義された zend_string は、HashTable に保存する際に複製されません。
