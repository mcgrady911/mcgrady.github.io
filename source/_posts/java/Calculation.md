# Calculation



### 反转字符串

```java
// StringBuffer的内置reverse方法进行逆序排序
public static String reverseStringBuilder(String s) {
    	StringBuilder sb = new StringBuilder(s);	                  
    	String afterreverse = sb.reverse().toString()；
	 return afterreverse;
}

// 递归
public static String reverse(String str){
    int len = str.length();
    if(len <= 1 )
        return str;

    String left = str.substring(0, len/2);
    String right = str.substring(len/2,len);

    return reverse(left) + reverse(right);
}

// CharArray
public static String reverseCharArray(String s) {
    char[] chars = s.toCharArray();
    String str = "";
    int len = chars.length;
    for (int i = len -1; i >= 0; i--) {
        str += chars[i];
    }

    return str;
}
```



### 求字符串中出现次数最多的字符

```java
public static char findMaxChar(String s) {
    char[] chars = s.toCharArray();
    int len = chars.length;
    Map<Character, Integer> map = new HashMap<>();
    for (int i = 0; i < len; i++) {
        if (map.containsKey(chars[i])) {
            map.put(chars[i], map.get(chars[i]) + 1);
        } else {
            map.put(chars[i], 1);
        }
    }

    Set<Map.Entry<Character, Integer>> entrySet = map.entrySet();
    Iterator<Map.Entry<Character, Integer>> iterator = entrySet.iterator();
    Character maxChar = iterator.next().getKey();
    int maxValue = map.get(maxChar);

    while (iterator.hasNext()) {
        Map.Entry<Character, Integer> temp = iterator.next();
        int count = temp.getValue();
        if (maxValue < count) {
            maxValue = count;
            maxChar = temp.getKey();
        }
    }

    return maxChar;
}
```

















































