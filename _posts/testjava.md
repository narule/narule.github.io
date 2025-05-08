





## 返回结果



```java
public int superpalindromesInRange(String L, String R) {
    long l = Long.valueOf(L);
    long r = Long.valueOf(R);
    int res = 0;
    for (long i = (long) Math.sqrt(l); i * i <= r; ) {        
        long p = nextPalindrome(i);
        if (p * p <= r && isPalindrome(p * p + "")) {            
            res++;      
        }       
        i = p + 1;   
    }    
    return res;
}
private long nextPalindrome(long l) {   
    String s = l + "";    
    int len = s.length();
    char[] chs = s.toCharArray();
    int i = (len - 1) / 2;   
    while (i >= 0 && chs[i] == '9') {      
        chs[i--] = '0';   
    }   
    if (i == -1) {      
        char[] res = new char[len + 1];        
        Arrays.fill(res, '0');      
        res[0] = res[len] = '1';       
        return Long.valueOf(new String(res));    
    } else {       
        chs[i]++;       
        int j = len - 1 - i;       
        while (i >= 0) {         
            chs[j++] = chs[i--];       
        }        
        return Long.valueOf(new String(chs));    
    }
}
private boolean isPalindrome(String s) {    
    int i = 0;   
    int j = s.length() - 1;   
    while (i < j) {
        if (s.charAt(i++) != s.charAt(j--)) {
            return false;       
        }   
    }return true;
}


```