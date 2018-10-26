---
title: Java-CPP-Pattern
key: 201810260
tags: Java CPP Pattern RegExp
---

# Java-CPP-Pattern

###  Java
   - `Pattern` class
      - `Pattern.compile(exp, tags)`构造
      - `.split(input, +/-0)`
      - `.matcher()` return Matcher
   - `Matcher` class
      - `.matches()` 全字符串匹配
      - `.lookingAt()` 无时间效应 => `reset(new Input String)`
      - `.find(fromIndex = 0)`有时间效应, `.regin([, ))`
      - 当`.lookingAt()`或`.find()`之后, 可使用
         - `.start(index = 0)`
         - `.end(index = 0)`
         - `.group(index = 0)`
      - `.repalceAll()`和`repalceFirst()`
      - `.appendReplacement()` => before find()
      - `.appendTail()` => after find()


###  CPP
- `std::regex` class 或 `std::wregex` class
   - `regex(std::string, std::regex::icase/nosubs/awk/grep)`构造
   - `.assign(nre, tags)`
   - `=`copy assignment
   - `.mark_count()` => 匹配项/匹配组的个数
   - `.flags()` => tags集合返回
- `std::regex_search(seq, std::regex, mft)` function
   - seq => `std::string, std::wstring, C style, C L style`
   - mft => `std::regex_constants::match_flag_type`
- `std::regex_search(seq, match, std::regex, mft)` function

   - match => `std::smatch, std::cmatch, std::wsmatch, std::wcmatch`

- `std::smatch` class 或 `std::cmatch, std::wsmatch, std::wcmatch` class
   - `.ready()` => 是否关联`std::regex`
   - `.size(), .empty()`
   - `.prefix()` => 到上一个匹配点, 配合`std::sregex_itera`使用, 不是匹配项
   - `.suffix()` => 后面所有字符串, 同上
   - `.length(n = 0)`
   - `.position(n = 0)`
   - `.str(n = 0)`
   - `.begin(), .end(), operator[]` => `ssub_match类`

- `std::ssub_match`class 或 `std::wssub_match, std::c/wc+sub_match` class
   - `.matched()` => 是否关联`std::regex`
   - `.first, .second`
   - `.length()`
   - `.str()`
   - `String operator()` => operator ITC

- **以下是极其常用的用法, 对`buffer`中输入字符串多次匹配点进行迭代**
- `std::sregex_iterator` class 或 `std::wsregex_iterator, std::c/wc+regex_iterator` class
   - `sregex_iterator([, ), std::regex)` 构造, `尾后构造`
   - `oparator*, operator->` => `std::smatch`
   - `operator++, --, operator==, !=`

- **正则匹配与替换结合**

- `regex_replace([, dst], seq, r, fmt = default, mft = default)`函数与`smatch.format([, dst], fmt, mft)`

   - fmt => regex replacement format string

   ```C++
    RegExp.lastMatch
    RegExp['$&']
    RegExp.$1-$9
    RegExp.input ($_)
    RegExp.lastParen ($+)
    RegExp.leftContext ($`)
    RegExp.rightContext ($')   
   ```

   - 关于mft => 一些匹配过程中的边角料要求
   
   ```C++
    inline constexpr match_flag_type match_default = 0;
    inline constexpr match_flag_type match_not_bol = /*unspecified*/;
    inline constexpr match_flag_type match_not_eol = /*unspecified*/;
    inline constexpr match_flag_type match_not_bow = /*unspecified*/;
    inline constexpr match_flag_type match_not_eow = /*unspecified*/;
    inline constexpr match_flag_type match_any = /*unspecified*/;
    inline constexpr match_flag_type match_not_null = /*unspecified*/;
    inline constexpr match_flag_type match_continuous = /*unspecified*/;
    inline constexpr match_flag_type match_prev_avail = /*unspecified*/;
    inline constexpr match_flag_type format_default = 0;
    inline constexpr match_flag_type format_sed = /*unspecified*/;
    inline constexpr match_flag_type format_no_copy = /*unspecified*/;
    inline constexpr match_flag_type format_first_only =/*unspecified*/;
    (since C++17)
   ```

- (写在最后) => {显然: C++更让人着迷, C++更美}