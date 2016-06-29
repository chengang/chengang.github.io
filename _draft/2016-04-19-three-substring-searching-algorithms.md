---
title: 3个在字符串中寻找子串的算法
date: 2016-04-19T11:55:55+00:00
layout: post
---
m是子串长度，n是父串长度。
  
θ是渐进，Ω是最快，O是最慢。

leetcode链接 <https://leetcode.com/problems/implement-strstr/>

## 1、暴力法

O(n*m)

<pre>int strLen(char *s) {
    int i = 0;
    while(*s != '\0') {
        s++;
        i++;
    }
    return i;
}

int strStr(char *haystack, char *needle) {
    if(*needle == '\0')
        return 0;

    int result = 1;
    int lenHaystack = strLen(haystack);
    int lenNeedle = strLen(needle);

    int ptrHaystack = 0;

    while(*(haystack + ptrHaystack) != '\0') {
        if(lenHaystack - ptrHaystack &lt; lenNeedle)
            break;

        if(*(haystack + ptrHaystack) != *needle) {
            ptrHaystack++;
            continue;
        }

        int ptrNeedle = 0;
        int isEq = 1;
        while(*(needle + ptrNeedle) != '\0') {
            if(*(needle + ptrNeedle) != *(haystack + ptrHaystack + ptrNeedle)) 
                isEq = 0;
            ptrNeedle++;
        }

        if(isEq) 
            return ptrHaystack;

        ptrHaystack++;
    }
    return -1;
}

</pre>

## 2、Rabin-Karp法

是在m上做文章。
  
预处理：求子串哈希。θ(m)
  
匹配：O(n*m)，θ(n)，因为有哈希，不用遍历子串

<pre>class Solution(object):
    # Rabin-Karp Algorthm
    def strStr(self, haystack, needle):
        """
        :type haystack: str
        :type needle: str
        :rtype: int
        """
        haystack_length = len(haystack)
        needle_length = len(needle)
        if haystack_length &lt; needle_length:
            return -1

        needle_hash = 0
        haystack_hash = 0
        for c in range(needle_length):
            needle_hash = needle_hash + ord(needle[c])
            haystack_hash = haystack_hash + ord(haystack[c])

        if haystack_hash == needle_hash and haystack[:needle_length] == needle:
            return 0

        for i in range(0, haystack_length - needle_length):
            haystack_hash = haystack_hash - ord(haystack[i]) + ord(haystack[i+needle_length])
            if needle_hash == haystack_hash: 
                if haystack[i+1:i+1+needle_length] == needle:
                    return(i + 1)

        return -1
        
s = Solution()
pos = s.strStr("mississippi","pi")
print(pos)
</pre>

## 3、Knuth-Morris-Pratt法

预处理：子串算出自己的next组。θ(m)
  
匹配：因为有next组，不用遍历父串。O(n)

<pre>class Solution(object):
    # Knuth-Morris-Pratt Algorthm
    def strStr(self, haystack, needle):
        """
        :type haystack: str
        :type needle: str
        :rtype: int
        """
        if not haystack and not needle:
            return 0
        if not haystack:
            return -1
        if not needle:
            return 0

        needle_index = 0
        haystack_index = 0
        needle_length = len(needle)
        haystack_length = len(haystack)


        seek_table = dict()
        seek_table[0] = -1
        seek_index_test = 0
        seek_index_patten = -1
        while seek_index_test &lt; needle_length:
            if seek_index_patten == -1 or needle[seek_index_test] == needle[seek_index_patten]:
                seek_table[seek_index_test] = seek_index_patten + 1
                seek_index_test += 1
                seek_index_patten += 1
            else:
                seek_index_patten = seek_table[seek_index_patten] - 1

        while haystack_index &lt; haystack_length:
            if haystack[haystack_index] == needle[needle_index]:
                if needle_index == needle_length - 1:
                    return haystack_index - needle_index
                else:
                    haystack_index += 1
                    needle_index += 1
            else:
                if needle_index == 0: 
                    haystack_index += 1
                elif seek_table[needle_index] == -1:
                    needle_index = 0
                else:
                    needle_index = seek_table[needle_index-1]

        return -1
        
s = Solution()
#pos = s.strStr("mississippi","pi1231231pi231")
#pos = s.strStr("bbbbababbbaabbba","abb")
#pos = s.strStr("mississippi","issip")
pos = s.strStr("abaaa","abb")
print(pos)

</pre>
