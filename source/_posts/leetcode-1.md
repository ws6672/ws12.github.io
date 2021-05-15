---
title: leetcode-1
date: 2020-07-02 10:42:14
tags:
---


### 1.前置知识

标号定义
```
ok:
for(int j=0;j<10;j++)
{
	System.out.println("i=" + i + ",j=" + j);
	if(j == 5) break ok;
}
```

定义数组
```
int[]result = new int[2];
```



### 2. 20-07-02


##### 2.1 [两数之和（简单）](https://leetcode-cn.com/problems/two-sum/)

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。


示例:
```
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

初始解法（暴力破解 O（n^2））：
```
-- 
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int i,j;
        int[]result = new int[2];
        for (i=0; i<nums.length-1; i++) {
            int temp = target-nums[i];
            for(j=i+1; j<nums.length; j++) {
                if (temp == nums[j]) {
                    result[0] = i;
                    result[1] = j;
                    i=nums.length-1;
                    j=nums.length;
                }
            }
        } 
        return result;
    }
}
```

基于hash的解法（存在唯一解，不考虑重复值）
```
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for(int i = 0; i< nums.length; i++) {
            if(map.containsKey(target - nums[i])) {
                return new int[] {map.get(target-nums[i]),i};
            }
            map.put(nums[i], i);
        }
        throw new IllegalArgumentException("No two sum solution");
    }
}	
```

##### 2.2 [整数反转(简单)](https://leetcode-cn.com/problems/reverse-integer/)


给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。

```
示例 1:

输入: 123
输出: 321


示例 2:

输入: -123
输出: -321


示例 3:


输入: 120
输出: 21
```

注意:
```
假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 [−231,  231 − 1]。请根据这个假设，如果反转后整数溢出那么就返回 0。
```

解析：`难点在于越界判定`，可以保留符号，一律转换为正数，每位转换后逆向操作，失败则越界设置为0，最后为结果加符号。


我的解法：
```
class Solution {
    public int reverse(int x) {
        int rs = 0,temp=0;
        // 提取符号
        int sy = x<0?-1:1;
        x = x<0?-x:x;

        do {
            temp = rs;
            int t = x%10;
            rs = rs*10+t;
            x/=10;
            //反向检验
            if((rs-t)/10!=temp) {
                rs=0;
                break;
            }
        } while(x != 0);
        rs*=sy;
        return rs;
    }
}
 
```


##### 2.3 [两数之和（中等）](https://leetcode-cn.com/problems/add-two-numbers/)

给出两个 非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 逆序 的方式存储的，并且它们的每个节点只能存储 一位 数字。

如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

示例：
```
输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807 
```

解法1：
```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode l = new ListNode(0);
        ListNode head = l,tl;
        do{
            int vl1=0,vl2=0;
            if(l1 != null) {
                vl1 = l1.val;
                l1 = l1.next;
            }
            if(l2 != null) {
                vl2 = l2.val;
                l2 = l2.next;
            }

            int temp = vl1+vl2+l.val;
            l.val = temp%10;
            l.next = new ListNode(temp/10);
            tl = l;
            l = l.next;
            
        }while(l1!=null||l2!=null);

        if(l.val == 0) {
            tl.next = null;
        }

        return head;
    }
}
```

### 3. 20-07-03

##### 3.1 [无重复字符的最长子串-中等](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

***描述如下***

给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。

```
示例 1:

输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
示例 2:

输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
示例 3:

输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。

请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。 
```

***通过滑动窗口解决问题***

在Tcp协议中，有滑动窗口这个概念，可以通过这个概念来处理这个问题。

首先，利用`Map<key,value>`存储数据，利用`KEY`的唯一性，来检测字符是否重复；key存储字符，value存储字符的下一位，重复时直接读取设置start即可。定义最长子串的区间[start,end]，end向后拓展，重复字符后重新设置start。

```
public static int lengthOfLongestSubstring(String s) {
	Map<Character,Integer> map = new HashMap<Character,Integer>();
	int length = 0;
	for(int end=0,start=0; end<s.length(); end++) {
		char cr = s.charAt(end);
		if (map.containsKey(cr)) {
			start = Math.max(map.get(cr), start);
		}
		length = Math.max(length, end-start+1);
		map.put(s.charAt(end), end+1);
	}
	return length;
}
```

##### 3.2 [回文数-简单](https://leetcode-cn.com/problems/palindrome-number/)


***描述如下***

判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

```
示例 1:

	输入: 121
	输出: true
示例 2:
	输入: -121
	输出: false
	解释: 从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。

示例 3:
	输入: 10
	输出: false
	解释: 从右向左读, 为 01 。因此它不是一个回文数。 
```


***解决方法***
回文数，通过计算倒置数字，比较即可；负数直接返回`FALSE`。

```
class Solution {
    public boolean isPalindrome(int x) {
        if(x<0) {
            return false;
        }
        int st=x,temp=0;

        do{
            temp=temp*10+x%10;
            x = x/10;
        }while(x!=0);

        return temp==st?true:false;
    }
}
```

##### 3.3 [加一-简单](https://leetcode-cn.com/problems/plus-one/)

***描述***

给定一个由整数组成的非空数组所表示的非负整数，在该数的基础上加一。

最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。

你可以假设除了整数 0 之外，这个整数不会以零开头。
```
示例 1:

输入: [1,2,3]
输出: [1,2,4]
解释: 输入数组表示数字 123。
示例 2:

输入: [4,3,2,1]
输出: [4,3,2,2]
解释: 输入数组表示数字 4321。
```


***解决方案***

存在进位就保持循环；如果最高位存在进位，那么重构数组

```
class Solution {
    public static int[] plusOne(int[] digits) {
        int i=digits.length-1;
        digits[i]++;
//        进位处理
        while (i>0) {
            if (digits[i]/10 >0) {
                digits[i-1]++;
            }
            digits[i] = digits[i]%10;
            i--;
        }
//        首位存在进位，重置数组
        if (digits[0]>9) {
            digits = new int[digits.length+1];
            digits[0]=1;
        }
        return digits;
    }
}
```



### 4. 20-07-03

##### 4.1 [罗马数字转整数（简单）](https://leetcode-cn.com/problems/roman-to-integer/)

***描述如下***

I             1
V             5
X             10

X             10
L             50
C             100

C             100
D             500
M             1000

```
罗马数字包含以下七种字符: I， V， X， L，C，D 和 M。

字符          数值
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
例如， 罗马数字 2 写做 II ，即为两个并列的 1。12 写做 XII ，即为 X + II 。 27 写做  XXVII, 即为 XX + V + II 。

通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做 IIII，而是 IV。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为 IX。这个特殊的规则只适用于以下六种情况：

+	I 可以放在 V (5) 和 X (10) 的左边，来表示 4 和 9。
+	X 可以放在 L (50) 和 C (100) 的左边，来表示 40 和 90。 
+	C 可以放在 D (500) 和 M (1000) 的左边，来表示 400 和 900。

给定一个罗马数字，将其转换成整数。输入确保在 1 到 3999 的范围内。

示例 1:

输入: "III"
输出: 3
示例 2:

输入: "IV"
输出: 4
示例 3:

输入: "IX"
输出: 9
示例 4:

输入: "LVIII"
输出: 58
解释: L = 50, V= 5, III = 3.
示例 5:

输入: "MCMXCIV"
输出: 1994
解释: M = 1000, CM = 900, XC = 90, IV = 4.

```



***解法如下***

```
class Solution {
    public int romanToInt(String s) {
        int sum = 0;

        for(int i=0; i<s.length();i++) {
            char cr = s.charAt(i);
            int temp = getValue(cr);
			//特殊值处理
            if(i!=s.length()-1) {
                int next = getValue(s.charAt(i+1));
                if(cr=='I'||cr=='X'||cr=='C') {
                    if(temp*5 == next || temp*10 == next) {
                        temp=next-temp;
                        i++;
                    }
                }
            }
            sum=sum+temp;
        }
        return sum;
    }

// 映射
    private int getValue(char cr) {
        switch (cr) {
            case 'I': return 1;
            case 'V': return 5;
            case 'X': return 10;
            case 'L': return 50;
            case 'C': return 100;
            case 'D': return 500;
            case 'M': return 1000;
            default: return 0;
        }

    }
}
```

##### 4.2 [最长公共前缀（简单）](https://leetcode-cn.com/problems/longest-common-prefix/)

***描述如下***

```
编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 ""。

示例 1:

输入: ["flower","flow","flight"]
输出: "fl"
示例 2:

输入: ["dog","racecar","car"]
输出: ""
解释: 输入不存在公共前缀。
说明:

所有输入只包含小写字母 a-z 。
```

***解法如下***

```
class Solution {
    public static String longestCommonPrefix(String[] strs) {
        int i=0;
        if(strs.length==0) {
            return "";
        }
        int leng = strs[0].length();
        for(String str:strs) {
            if(leng>str.length()) {
                leng = str.length();
            }
        }
        if(leng==0) {
             return "";
        }


        for(;i<leng;i++) {
            char cr = strs[0].charAt(i);
            for(String str:strs) {
                if(cr != str.charAt(i)) {
                    return strs[0].substring(0, i);
                }
            }
        }
        return strs[0].substring(0, i);
    }
}
```