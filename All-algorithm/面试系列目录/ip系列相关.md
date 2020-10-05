###   复原IP地址

####  题目（93-中等）

给定一个只包含数字的字符串，复原它并返回所有可能的 IP 地址格式。

有效的 IP 地址 正好由四个整数（每个整数位于 0 到 255 之间组成，且不能含有前导 0），整数之间用 '.' 分隔。

例如："0.1.2.201" 和 "192.168.1.1" 是 有效的 IP 地址，但是 "0.011.255.245"、"192.168.1.312" 和 "192.168@1.1" 是 无效的 IP 地址。

示例 1：

输入：s = "25525511135"
输出：["255.255.11.135","255.255.111.35"]
示例 2：

输入：s = "0000"
输出：["0.0.0.0"]
示例 3：

输入：s = "1111"
输出：["1.1.1.1"]
示例 4：

输入：s = "010010"
输出：["0.10.0.10","0.100.1.0"]
示例 5：

输入：s = "101023"
输出：["1.0.10.23","1.0.102.3","10.1.0.23","10.10.2.3","101.0.2.3"]

####  思考

`DFS`+ 回溯

####  代码

```java
 List<String> res = new LinkedList<>();
  LinkedList<String> segment = new LinkedList<>();
   public  List<String> restoreIpAddresses(String s) {
        helper(s, 0);
        return res;
    }
 void helper(String s, int start){
        if(start == s.length() && segment.size() == 4){
            StringBuilder t = new StringBuilder();
            for (String se: segment) t.append(se).append(".");
             t.deleteCharAt(t.length() - 1);
            res.add(t.toString());
            return;
        }
        if(start < s.length() && segment.size() == 4)return;
        for(int i = 1;i <= 3; i++){
            if(start + i - 1 >= s.length())return;
            if(i != 1 && s.charAt(start) == '0')return;
            String str = s.substring(start, start + i);
            if(i == 3 && Integer.parseInt(str) > 255)return;
            segment.add(str);
            helper(s, start + i);
            segment.removeLast();
        }
    }
```

###  验证是否是IP地址

####  题目（468-中等）

编写一个函数来验证输入的字符串是否是有效的 `IPv4` 或 `IPv6` 地址。

如果是有效的 `IPv4` 地址，返回 "`IPv4`" ；
如果是有效的 `IPv6` 地址，返回 "`IPv6`" ；
如果不是上述类型的 IP 地址，返回 "Neither" 。
`IPv4` 地址由十进制数和点来表示，每个地址包含 4 个十进制数，其范围为 0 - 255， 用(".")分割。比如，172.16.254.1；

同时，`IPv4` 地址内的数不会以 0 开头。比如，地址 172.16.254.01 是不合法的。

`IPv6` 地址由 8 组 16 进制的数字来表示，每组表示 16 比特。这些组数字通过 (":")分割。比如,  `2001:0db8:85a3:0000:0000:8a2e:0370:7334` 是一个有效的地址。而且，我们可以加入一些以 0 开头的数字，字母可以使用大写，也可以是小写。所以， `2001:db8:85a3:0:0:8A2E:0370:7334` 也是一个有效的 `IPv6` address地址 (即，忽略 0 开头，忽略大小写)。

然而，我们不能因为某个组的值为 0，而使用一个空的组，以至于出现 (::) 的情况。 比如， `2001:0db8:85a3::8A2E:0370:7334` 是无效的 `IPv6` 地址。

同时，在 `IPv6` 地址中，多余的 0 也是不被允许的。比如， `02001:0db8:85a3:0000:0000:8a2e:0370:7334` 是无效的。

示例 1：

```JAVA
输入：IP = "172.16.254.1"
输出："IPv4"
解释：有效的 IPv4 地址，返回 "IPv4"
```

示例 2：

```java
输入：IP = "2001:0db8:85a3:0:0:8A2E:0370:7334"
输出："IPv6"
解释：有效的 IPv6 地址，返回 "IPv6"
```

示例 3：

```java
输入：IP = "256.256.256.256"
输出："Neither"
解释：既不是 IPv4 地址，又不是 IPv6 地址
```

示例 4：

```java
输入：IP = "2001:0db8:85a3:0:0:8A2E:0370:7334:"
输出："Neither"
```

####  思想

见代码

####  代码

```java
private static String validIPAddress(String IP) {
        String  [] ipV4 = IP.split("\\.",-1);
        if(ipV4.length == 4){
            return  isIPv4(ipV4);
        }
        String [] ipV6 = IP.split(":",-1);
        if(ipV6.length == 8){
            return  isIPv6(ipV6);
        }
        return  "Neither";
    }
    private  static  String isIPv4(String [] v4){
        /**
         *
         */
        for(String p: v4){
            if(p.length()>3 || p.length()<=0)
            {
                return   "Neither";
            }
            for(int i =0;i<p.length();i++){
                if(!Character.isDigit(p.charAt(i))){
                    return   "Neither";
                }
            }
            int num = Integer.parseInt(p);
            if(num>255 || String.valueOf(num).length() !=p.length())
                return   "Neither";
        }
        return  "IPv4";
    }
    private  static  String isIPv6(String [] v6){
        for(String q: v6){
            if(q.length()>4 ||q.length()<1){
                return   "Neither";
            }
            for(int i =0;i<q.length();i++){
                char c = q.charAt(i);
                if(!Character.isDigit(c) && !( 'a' <= c && c <= 'f') && !('A' <= c && c <= 'F')){
                    return "Neither";
                }
            }
        }
        return  "IPv6";
    }
```

