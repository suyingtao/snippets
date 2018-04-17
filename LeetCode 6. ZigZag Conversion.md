## 问题描述
The string "PAYPALISHIRING" is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)

    P   A   H   N
    A P L S I I G
    Y   I   R
And then read line by line: "PAHNAPLSIIGYIR"

Write the code that will take a string and make this conversion given a number of rows:

string convert(string s, int numRows);
##### Example 1:

    Input: s = "PAYPALISHIRING", numRows = 3
    Output: "PAHNAPLSIIGYIR"
##### Example 2:

    Input: s = "PAYPALISHIRING", numRows = 4
    Output: "PINALSIGYAHRPI"
    Explanation:
     
    P     I    N
    A   L S  I G
    Y A   H R
    P     I


## 思路

    建立一个numRows大小的数组rows。
    遍历字符串s。
    将s[i]依次加入到rows[从0到numRows-1再到0......]。
    最后rows数组各项连接起来即答案。
    时间复杂度O(n)

## JS代码

```JS
/**
 * @param {string} s
 * @param {number} numRows
 * @return {string}
 */
var convert = function(s, numRows) {
    if(numRows === 1) return s;
    
    let result = '';
    let rows = [];
    for(let i = 0; i < numRows; i++) {
        rows[i] = '';
    }
    for(let i = 0, n = 0, add = true; i < s.length; i++) {
        rows[n] += s[i];

        if(n === numRows - 1) add = false;
        
        if(n === 0) add = true;
        
        if(add) {
            n++;
        } else {
            n--
        }
        
    }
    for(let i = 0; i < numRows; i++) {
        result += rows[i];
    }
    return result;
};

convert("PAYPALISHIRING",3); // PAHNAPLSIIGYIR
```
