## Constraints
### Logical constraints
- Longest Repeating Character Replacement (sliding window)
  - You can replace `k` characters into anything else. Find longest sequence of same characters.
  - L,R are window pointers. R always progresses by 1 each round
    - After every R increment, there is some logic to maintain the "validity" of L, namely
      that the window itself is always shorter than k + "max freq"
    - As L increments, str[L] is removed from occurrence counting table
    - As R increments, str[R] is added to the occurrence counting table
  - Intuitively, "max freq" should be re-scanned every time to match the frequency
  - However, it can be shown that "max freq" never needs to be decremented, only incremented, meaning it is only really a heuristic
    - Instead of thinking of this as a variable size window, think of this as a fixed sized window but with a fixed size of `k + max_freq`. If `max_freq` is updated at any particular time, we treat that as having one free +1 to allowed window size, regardless of what is in the window
    - Now, we only need to validate that `max_freq` is only updated when needed.
      - Initially, there are nothing in the window. Say that we have character `a` that occurred once, max_freq = 1, then we can allow `k` more characters other than the `a`
      - If `a` occurs again before reading in `>k` more characters (`abca` for `k=2`), we know that `abca` is a valid window
      - Now, if `abcad`, then `5 - 2 > 2` and we will need to remove from the window, note that as we remove the recorded count of `a` is decremented, meaning max_freq is practically locked at 2 until something better shows up in a valid window down the line
      - Effectively, every `max_freq` update means a valid window of "some size smaller than `max_freq + k`" can be constructed
  - Consider next that upon every `R` increment, the best length is updated to `R-L+1`
    - Since the `L` incrementing loop makes sure `R-L+1 < max_freq+k`, it is not possible for later "invalid" results to surpass previous valid results.
    - The previous valid result is a logical conclusion of `max_freq`, since `max_freq` stays the same the validity of the window length constraint stays the same
  - The guiding principle is **if updates are never invalidated, just try to run all possible updates without worrying about correctness of intermediate state**
  - Technically, scanning all alphabets or not has no impact to the theoretical runtime since `O(26n) = O(n)`, but practically speaking the non-scanning solution will always be faster

  > Sample explanation online:
  > Since we are only interested in the longest valid substring, our sliding windows need not shrink, even if a window may cover an invalid substring. We either grow the window by appending one char on the right, or shift the whole window to the right by one. And we only grow the window when the count of the new char exceeds the historical max count (from a previous window that covers a valid substring).
  > That is, we do not need the accurate max count of the current window; we only care if the max count exceeds the historical max count; and that can only happen because of the new char.
  > Here's my implementation that's a bit shorter
  > ```java
  > // https://leetcode.com/problems/longest-repeating-character-replacement/solutions/91271/java-12-lines-o-n-sliding-window-solution-with-explanation/comments/95833
  > public int characterReplacement(String s, int k)
  > {
  >    int[] count = new int[128];
  >    int max=0;
  >    int start=0;
  >    for(int end=0; end<s.length(); end++)
  >    {
  >        max = Math.max(max, ++count[s.charAt(end)]);
  >        if(max+k<=end-start)
  >            count[s.charAt(start++)]--;
  >    }
  >    return s.length()-start;
  > }
  > ```

### Numeric constraints
- Two-sum two-pointer solution (sorted array)
  - If L+R < target, then no need to move R further to the left since that only makes the target smaller
  - If L+R > target, the no need to move L to the right since that only makes the target larger
  - For a new value of L (after moving right), there's no need for R to be reset, since any value right of R will go over
  - (Mirrored proof for not resetting L)
  - Never need to move R past L
- Longest substring without duplicate characters
  - Sliding window technique --> when the first duplicate is reached, no need to keep checking since all other substrings will contain this initial substring with duplicates in it
  - Dumb solution:
    - On every duplicate reached, migrate the old occurrence table to the new by disregarding all characters last occurred before the prev occurence of the duplicate
      - *ex.* `abc -> {a:0 b:1 c:2}`, upon seeing `abca` we create new table containing `{b:1, c:2, a:3}`. This loops over existing stuff and could be bothersome
  - Smarter solution:
    - Keep reusing the same occurrence table, and use a set instead of map. We only really care about a character being there and not where it is there
      - For small character sets defined by the problem, such as ascii, we can just use an array
    - *ex.* `abc -> {a, b, c}`, upon seeing `abca` we keep sliding the window to the right and removing characters one by one, since they're uniquely identified anyway
      - `a|bc,a: {a,b,c} -> {b, c} (remove a) -> {b, c, a} (add a back in)`
      - Consider `abc,b: {a, b, c}, {b, c}, {c}, {c, b}`. We keep sliding until the new character is no longer in the set.

## Skips
- 3 Sum hashmap vs. sorted array (+two pointer) solution
  - Problem: find all triplets in an unsorted array that adds to 0, no duplicate indices
  - (For simplicity, ignore 0's in this example!)
  - Hashmap: create a set of positive numbers and negative numbers (O(N)). Brute force all pairs in the negative number set, and match their sums against the positive number set (O(N^2)), then do the same in reverse. O(N) space.
  - Observation: since this is O(N^2) anyway, sort the array and see if any optimizations could happen!
  - Two pointer: sort the array
    - For each a_i, run two_sum on the right of i to find the matching sum. This is O(N) per loop and O(N^2) in general
    - If a_i is equal to a_(i-1), we can just skip the iteration since we already found the triplets corresponding to a_i
    - Note that during two-pointer, we actually want to keep marching after the first match
      - The right pointer should only be decremented once, while the left pointer keeps getting incremented
      - (The reverse is also valid, as long as only one pointer keeps moving!)
- Water bucket: water amount = index difference * min(height[i], height[j])
  - Do two-pointer, switch between perspectives whenever the "iterated-upon side" becomes the limiting height
  - Two pointer is natural to this problem because we want to maximize the distance between columns
