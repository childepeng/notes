# 编辑距离算法（Edit distance）
编辑距离算法也叫Levenshtein distance，原理就是计算一个字符串转换为另一个字符串的编辑次数，编辑操作包括删除、插入、替换；编辑次数越小，相似度越高。
编辑距离可通过矩阵算出来，具体推算过程如下：
1. 根据两段文本构建矩阵，比如：S1="abcd",S2="ebgdh"，矩阵为S[i,j] = S[S1.length+1, S2.length+1]
    * 如果：i==0，则：S[0, j]=j; 
    * 如果：j==0，则：S[i, 0]=i;
    * 如果：i>0 且 j>0，则：S[i, j]=min( S[i-1,j]+1, S[i,j-1]+1, S[i-1,j-1]+X );  min表示取最小值，X值参考下一条规则
    * 如果：S1[i]==S2[j]，则：X=0；否则：X=1 
    ![](http://static.laop.cc/images/editdistance.png)
2. 根据规则，矩阵最后一个值为 S1 -> S2的编辑距离。
3. 相似度：similaryDegree = 1 - edts / Math.max(s1.length(), s2.length());

Java实现如下
```
    /**
     * 编辑距离算法，一个字符串转换为另一个字符串最少的编辑次数，编辑包括字符插入、删除、替换
     *
     * @param source
     * @param target
     * @return
     */
    public static int levenShtein(String source, String target) {
        char sourceChars[] = source.toCharArray(), targetChars[] = target.toCharArray();
        int sourceLen = sourceChars.length, targetLen = targetChars.length;
        int tmp[][] = new int[sourceLen + 1][targetLen + 1];

        for (int i = 0; i < sourceLen; i++) {
            tmp[i + 1][0] = i + 1;
            for (int j = 0; j < targetLen; j++) {
                if (i == 0) {
                    tmp[0][j + 1] = j + 1;
                }

                int t = tmp[i][j + 1] + 1;
                int l = tmp[i + 1][j] + 1;
                int m = 0;
                if (sourceChars[i] == targetChars[j]) {
                    m = tmp[i][j];
                } else {
                    m = tmp[i][j] + 1;
                }
                tmp[i + 1][j + 1] = Math.min(t, Math.min(l, m));
            }
        }
        return tmp[sourceLen][targetLen];
    }

    public static void main(String[] args) {
        String s1 = "abc";
        String s2 = "dbd";
        int dit = levenShtein(s1, s2);
        
        // 相似度
        float similaryDegree = 1 - (float) dit / Math.max(s1.length(), s2.length());
    }
```


# 最长公共子序列
通过计算两个段文本中最长公共子序列长度判断相似度。比如：abcdef与poicdeg的公共子序列为cde，长度为3。
算法可通过矩阵实现，矩阵推导过程如下：

1. 根据两段文本构建矩阵，比如：S1="abcdefg", S2="thjabcfg"，矩阵S[S1.length, S2.length]
    * 如果：S1[i]!=S2[j]，则：S[i,j]=0
    * 如果：S1[i]==S2[j]，则：S[i,j]=S[i-1,j-1]+1
    ![](http://static.laop.cc/images/lcs.png)
2. 矩阵中最大值为最长公共子序列长度。
3. 相似度：similaryDegree = commonLength / Math.max(s1.length(), s2.length());

Java实现如下
```
    /**
     * 最长公共序列长度
     *
     * @param source
     * @param target
     * @return
     */
    public static int longestCommonLength(String source, String target) {
        char[] sourceChars = source.toCharArray(), targetChars = target.toCharArray();
        int sourceLen = sourceChars.length, targetLen = targetChars.length;
        int tmp[][] = new int[sourceLen][targetLen];
        int max = 0;

        for (int i = 0; i < sourceLen; i++) {
            for (int j = 0; j < targetLen; j++) {
                if (sourceChars[i] == targetChars[j]) {
                    if (i == 0 || j == 0) {
                        tmp[i][j] = 1;
                    } else {
                        tmp[i][j] = tmp[i - 1][j - 1] + 1;
                    }
                }
                if (max < tmp[i][j]) {
                    max = tmp[i][j];
                }
            }
        }
        return max;
    }

    public static void main(String[] args) {
        String s1 = "abc";
        String s2 = "dbd";
        int commonLength = longestCommonLength(s1, s2);

        // 相似度
        float similaryDegree = (float) commonLength / Math.max(s1.length(), s2.length());
    }

```

# 余弦相似度算法
余弦相似度算法：通过计算两个特征向量的空间夹角，判断相似度。
- 通过分词，创建空间向量模型，并得出待比较对象的特征向量；
- 根据余弦公式，计算余弦，值越小，相似度越低，当等于1时，表示相等

余弦公式：
![](http://static.laop.cc/images/cosine.png)

```
    public static double cosineSimilary(String source, String target) {
        //创建向量空间模型，使用map实现，主键为词项，值为长度为2的数组，
        //存放着对应词项在字符串中的出现次数
        Map<String, int[]> vectorSpace = new HashMap<>();

        //以空格为分隔符，分解字符串
        String strArray[] = source.split(" ");
        for (int i = 0; i < strArray.length; ++i) {
            if (vectorSpace.containsKey(strArray[i])) {
                vectorSpace.get(strArray[i])[0]++;
            } else {
                int[] temp = new int[2];
                temp[0] = 1;
                vectorSpace.put(strArray[i], temp);
            }
        }

        strArray = target.split(" ");
        for (int i = 0; i < strArray.length; ++i) {
            if (vectorSpace.containsKey(strArray[i]))
                ++(vectorSpace.get(strArray[i])[1]);
            else {
                int[] temp = new int[2];
                temp[1] = 1;
                vectorSpace.put(strArray[i], temp);
            }
        }

        //计算相似度
        double vector1Modulo = 0.00;//向量1的模
        double vector2Modulo = 0.00;//向量2的模
        double vectorProduct = 0.00; //向量积
        Iterator iter = vectorSpace.entrySet().iterator();
        while (iter.hasNext()) {
            Map.Entry entry = (Map.Entry) iter.next();
            int[] temp = (int[]) entry.getValue();
            vector1Modulo += temp[0] * temp[0];
            vector2Modulo += temp[1] * temp[1];
            vectorProduct += temp[0] * temp[1];
        }

        vector1Modulo = Math.sqrt(vector1Modulo);
        vector2Modulo = Math.sqrt(vector2Modulo);

        //返回相似度
        return (vectorProduct / (vector1Modulo * vector2Modulo));
    }
```
