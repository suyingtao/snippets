# 12. Integer to Roman  
Roman numerals are represented by seven different symbols: I, V, X, L, C, D and M.

    Symbol       Value
    I             1
    V             5
    X             10
    L             50
    C             100
    D             500
    M             1000
For example, two is written as II in Roman numeral, just two one's added together. Twelve is written as, XII, which is simply X + II. The number twenty seven is written as XXVII, which is XX + V + II.

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not IIII. Instead, the number four is written as IV. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as IX. There are six instances where subtraction is used:

I can be placed before V (5) and X (10) to make 4 and 9. 
X can be placed before L (50) and C (100) to make 40 and 90. 
C can be placed before D (500) and M (1000) to make 400 and 900.
Given an integer, convert it to a roman numeral. Input is guaranteed to be within the range from 1 to 3999.  


## my solution
```JS
var intToRoman = function(num) {
    var str = 'MDCLXVI';
    var nums = [1000, 500, 100, 50, 10, 5, 1];
    var result = '';
    for(var i = 0; i < str.length; i++) {
        var tempN = Math.floor(num / nums[i]);
        num = num % nums[i];
        var tempS = Array(1 + tempN).join(str[i]);
        if(i < 6) {
            var subtractIndex = i % 2 === 0 ? (i + 2) : (i + 1);
            if((num + nums[subtractIndex]) >= nums[i]) {
                tempS = tempS.concat(str[subtractIndex] + str[i]);
                num = num - nums[i] + nums[subtractIndex];
            }
        }
               
        result = result.concat(tempS)
    }
    return result;
};
```  

## a nice solution
```JS
/**
 * @param {number} num
 * @return {string}
 */
var intToRoman = function(num) {
    romanHashReverse = {
        'M':1000,
        'CM':900,
        'D':500,
        'CD':400,
        'C':100,
        'XC':90,
        'L':50,
        "XL":40,
        'X':10,
        "IX":9,
        'V':5,
        'IV':4,
        'I':1
    }
    var roman = ''
    for(var key in romanHashReverse ){
      
       while(num >= romanHashReverse[key]){
            roman += key
            num -= romanHashReverse[key]
       }  
    }
    
    return roman
    
};
```
