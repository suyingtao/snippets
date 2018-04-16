## 问题描述  

Given a string s, find the longest palindromic substring in s. You may assume that the maximum length of s is 1000.   

Example 1:

    Input: "babad"
    Output: "bab"
    Note: "aba" is also a valid answer.  

Example 2:

    Input: "cbbd"
    Output: "bb"  


## 思路

    回文串分为奇数和偶数两种情况。
    遍历字符串s，找中心点，分为两种情况：
    指针指向的字符当做回文串的中心点，
    指针指向的字符和前一个字符合起来当做中心点。
    找好中心点后分别向左向右延伸，使temp字符串长度大于当前找到的最长回文串长度。
    做以下循环：
    1. 如果temp字符串不是回文串，那么退出：
    1. 记录temp、temp长度。
    2. 向左右扩大temp。

## JS代码
```JS
/**
 * @param {string} s
 * @return {string}
 */
var longestPalindrome = function(s) {
    let max = 1;
    let result = s[0];
    for(let i = 1; i < s.length; i++) {
        // s[i] is center
        let r = Math.ceil(max / 2);
        while(i - r >= 0 && i + r < s.length) {
            let tempStr = s.slice(i - r, i + r + 1);
            if(palindrome(tempStr)) {
                result = tempStr;
                max = 1 + 2 * r;
                r ++;
            } else {
                break;
            }
        }
        // s[i-1] add s[i] is center
        while(i - r >= 0 && i + r - 1 < s.length) {
            let tempStr = s.slice(i - r, i + r);
            if(palindrome(tempStr)) {
                result = tempStr;
                max = 2 * r;
                r ++;
            } else {
                break;
            }
        }
    }
    return result;
};

function palindrome(s) {
    if(s.length === 1) return true;

    for(let i = 0, l = s.length - 1; i < l; i ++, l --) {
        if(s[i] !== s[l]) {
            return false;
        }
    }

    return true;
}

longestPalindrome('babac'); // 'bab'


```
