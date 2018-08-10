---
title: NSDateFormatter 时间格式
date: 2018-03-22 15:35:49
tags: NSDateFormatter
categories: foundation
toc: true
---

## 时间匹配格式

<caption><a>Date Field Symbol Table</a></caption>

| Field | Sym. | No. | Example | Description |
| --- | --- | --- | --- | --- |
| era | G | 1..3 | AD | Era - Replaced with the Era string for the current date. One to three letters for the abbreviated form, four letters for the long form, five for the narrow form. |
||| 4 | Anno Domini |
||| 5 | A |
| year | y | 1..n | 1996 | Year. Normally the length specifies the padding, but for two letters it also specifies the maximum length. Example:
|| Y | 1..n | 1997 | Year (in "Week of Year" based calendars). This year designation is used in ISO year-week calendar as defined by ISO 8601, but can be used in non-Gregorian based calendar systems where week date processing is desired. May not always be the same value as calendar year. |
|| u | 1..n | 4601 | Extended year. This is a single number designating the year of this calendar system, encompassing all supra-year fields. For example, for the Julian calendar system, year numbers are positive, with an era of BCE or CE. An extended year value for the Julian calendar system assigns positive values to CE years and negative values to BCE years, with 1 BCE being year 0. |
| quarter | Q | 1..2 | 02 | Quarter - Use one or two for the numerical quarter, three for the abbreviation, or four for the full name. |
||| 3 | Q2 |
||| 4 | 2nd quarter |
|| q | 1..2 | 02 | **Stand-Alone** Quarter - Use one or two for the numerical quarter, three for the abbreviation, or four for the full name. |
||| 3 | Q2 |
||| 4 | 2nd quarter |
| month | M | 1..2 | 09 | Month - Use one or two for the numerical month, three for the abbreviation, or four for the full name, or five for the narrow name. |
||| 3 | Sept |
||| 4 | September |
||| 5 | S |
|| L | 1..2 | 09 | **Stand-Alone** Month - Use one or two for the numerical month, three for the abbreviation, or four for the full name, or 5 for the narrow name. |
||| 3 | Sept |
||| 4 | September |
||| 5 | S |
|| l | 1 | * | Special symbol for Chinese leap month, used in combination with M. Only used with the Chinese calendar. |
| week | w | 1..2 | 27 | Week of Year. |
|| W | 1 | 3 | Week of Month |
| day | d | 1..2 | 1 | Date - Day of the month |
|| D | 1..3 | 345 | Day of year |
|| F | 1 | 2| Day of Week in Month. The example is for the 2nd Wed in July |
|| g | 1..n | 2451334 | Modified Julian day. This is different from the conventional Julian day number in two regards. First, it demarcates days at local zone midnight, rather than noon GMT. Second, it is a local number; that is, it depends on the local time zone. It can be thought of as a single number that encompasses all the date-related fields. |
| week day | E | 1..3 | Tues | Day of week - Use one through three letters for the short day, or four for the full name, or five for the narrow name. |
||| 4 | Tuesday |
||| 5 | T |
|| e | 1..2 | 2 | Local day of week. Same as E except adds a numeric value that will depend on the local starting day of the week, using one or two letters. For this example, Monday is the first day of the week. |
||| 3 | Tues |
||| 4 | Tuesday |
||| 5 | T |
|| c | 1 | 2 | **Stand-Alone** local day of week - Use one letter for the local numeric value (same as 'e'), three for the short day, or four for the full name, or five for the narrow name.  |
||| 3 | Tues |
||| 4 | Tuesday |
||| 5 | T |
| period | a | 1 | AM | AM or PM |
| hour | h | 1..2 | 11 | Hour [1-12].  |
|| H | 1..2 | 13 | Hour [0-23]. |
|| K | 1..2 | 0 | Hour [0-11]. |
|| k | 1..2 | 24 | Hour [1-24]. |
|| j | 1..2 | n/a | This is a special-purpose symbol. It must not occur in patterns, but is reserved for use in APIs doing flexible date pattern generation, and requests the preferred format (12 vs 24 hour) for the language in question. |
| minute | m | 1..2 | 59 | Minute. Use one or two for zero padding. |
| second | s | 1..2 | 12 | Second. Use one or two for zero padding. |
|| S | 1..n | 3457 | Fractional Second - rounds to the count of letters. (example is for 12.34567) |
|| A | 1..n | 69540000 | Milliseconds in day. This field behaves  _exactly_  like a composite of all time-related fields, not including the zone fields. As such, it also reflects discontinuities of those fields on DST transition days. On a day of DST onset, it will jump forward. On a day of DST cessation, it will jump backward. This reflects the fact that is must be combined with the offset field to obtain a unique local time value. |
| zone | z | 1..3 | PDT _fallbacks: _HPG-8:00 GMT-08:00| Timezone - with the s _pecific non-location format_ . Where that is unavailable, falls back to  _localized GMT format_ . Use one to three letters for the short format or four for the full format. In the short format, metazone names are not used unless the commonlyUsed flag is on in the locale.|
||| 4 | Pacific Daylight Time _fallbacks:_  HPG-8:00 GMT-08:00|
|| Z | 1..3 | -0800 | Timezone - Use one to three letters for RFC 822 format, four letters for the localized GMT format.|
||| 4 | HPG+8:00 _fallbacks:_ GMT-08:00 |
|| v | 1 | PT | Timezone - with the  _generic_   _non-location format_ . Where that is unavailable, uses special fallback rules given in  _[Appendix J][6]_ . Use one letter for short format, four for long format.|
||| 4 | Pacific Time _fallbacks:_ Pacific Time (Canada) Pacific Time (Yellowknife) United States (Los Angeles) Time HPG-8:35 GMT-08:35|
|| V | 1 | PST HPG-8:00 GMT-08:00 | Timezone - with the same format as z, except that metazone timezone abbreviations are to be displayed if available, regardless of the value of commonlyUsed.|
||| 4 | United States (Los Angeles) Time _fallbacks:_  HPG-8:35 MT-08:35 | Timezone - with the  _generic_   _location format_ . Where that is unavailable, falls back to the localized GMT format. (Fallback is only necessary with a GMT-style Timezone ID, like Etc/GMT-830.) This is especially useful when presenting possible timezone choices for user selection, since the naming is more uniform than the v format.|

## 参考

1. [UNICODE LOCALE DATA MARKUP LANGUAGE (LDML)](http://unicode.org/reports/tr35/tr35-10.html#Date_Format_Patternss)
  





