## String





### indexOf(String str)

```java
/**
  * Code shared by String and StringBuffer to do searches. The
  * source is the character array being searched, and the target
  * is the string being searched for.
  *
  * @param   source       the characters being searched.	原字符串，toCharArray
  * @param   sourceOffset offset of the source string.	原字符串起始下标
  * @param   sourceCount  count of the source string.	原字符串长度
  * @param   target       the characters being searched for.	目标字符串，toCharArray
  * @param   targetOffset offset of the target string.	目标字符串起始下标
  * @param   targetCount  count of the target string.	目标字符串长度
  * @param   fromIndex    the index to begin searching from.	开始查询下标
*/
static int indexOf(char[] source, int sourceOffset, int sourceCount,
            char[] target, int targetOffset, int targetCount,
            int fromIndex) {
    if (fromIndex >= sourceCount) {
        return (targetCount == 0 ? sourceCount : -1);
    }
    if (fromIndex < 0) {
        fromIndex = 0;
    }
    /* 如果目标str长度为0 */
    if (targetCount == 0) {
        return fromIndex;
    }
	/* 目标字符串首字符 */
    char first = target[targetOffset];
    /* 由于只需要查询起始下标，故只需遍历source.length - target.length */
    int max = sourceOffset + (sourceCount - targetCount);

    for (int i = sourceOffset + fromIndex; i <= max; i++) {
        /* Look for first character. */
        /* 查询第一次出现首字符 */
        if (source[i] != first) {
            while (++i <= max && source[i] != first);
        }

        /* Found first character, now look at the rest of v2 */
        /* 从首字符之后，查询之后字符是否符合 */
        if (i <= max) {
            int j = i + 1;
            int end = j + targetCount - 1;
            for (int k = targetOffset + 1; j < end && source[j]
                 == target[k]; j++, k++);

            /* 由于end = j + targetCount - 1,因此只需j == end，即查到目标字符最后一个相等，则找到 */
            if (j == end) {
                /* Found whole string. */
                return i - sourceOffset;
            }
        }
    }
    return -1;
}
```

