# 3164. Find the Number of Good Pairs II (Factors + Hashmap) [link](https://leetcode.com/problems/find-the-number-of-good-pairs-ii/description/)

![alt text](image.png)

```java
class Solution {
    public long numberOfPairs(int[] nums1, int[] nums2, int k) {
        Map<Integer, Integer> map = new HashMap<>();

        // Store nums2[j] * k
        for (int x : nums2) {
            int val = x * k;
            map.put(val, map.getOrDefault(val, 0) + 1);
        }

        long count = 0;

        // Check divisors of nums1[i]
        for (int x : nums1) {
            for (int d = 1; d * d <= x; d++) {
                if (x % d == 0) {
                    if (map.containsKey(d))
                        count += map.get(d);

                    int other = x / d;
                    if (other != d && map.containsKey(other))
                        count += map.get(other);
                }
            }
        }
        return count;
    }
}
```