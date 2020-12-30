---
layout: default
permalink: /assignment/clojure-koans/
title:
---
# Clojure Koans

## Assignment Criteria
<ul>
<li> Apply the functional style of programming to a range of simple problems</li>
<li>Implement a complex application in a functional language</li>
<li>Evaluate the merits of functional programming</li>
</ul>
## Basic Koans

Replace the blanks with the code which is necessary to solve the problem.



{% highlight clojure %}
;1. You cannot generally float to heavens of integers: 
    (= __ (= 2 2.0))
    (= false (= 2 2.0))
    
;2. A looser equality is possible: 
    (== 2.0 2 __)
    (== 2 2.0 02) 
    
;3. You can use a list like a stack to get the first element: 
   (= __ (peek '(:a :b :c :d :e))) 
   (= :a (peek '(:a :b :c :d :e))) 
    
;4. Or the others: 

   (= __ (pop '(:a :b :c :d :e))) 
   (= '(:b :c :d :e)(pop '(:a :b :c :d :e))) 
    
;5. You can get the value of any item using its index: 
   (= __ (nth [:peanut :butter :and :jelly] 3)) 
   (= :jelly (nth [:peanut :butter :and :jelly] 3)) 
    
;6. You can slice a vector: 
   (= __ (subvec [:peanut :butter :and :jelly] 13)) 
   (= [:butter :and] (subvec [:peanut :butter :and :jelly] 1 3)) 
    
;7. You can ask clojure for the union of two sets: 
    (= __ (clojure.set/union #{1 2 3 4} #{2 3 5})) 
    (= #{1 2 3 4 5} (clojure.set/union #{1 2 3 4} #{2 3 5})) 
 
;8. In a map you can find out if a key is present: 
    (= __ (contains? {:a nil :b nil} :b)) 
    (= (not (contains? {:a nil :b nil} :c)) (contains? {:a nil :b nil} :b)) 
 
;9. Often you will need to get the keys of a map: 
    (= (list ______ ) (sort (keys {2006 "Torino" 2010 "Vancouver" 2014 "Sochi"}))) 
    (= (list 2006 2010 2014)(sort (keys {2006 "Torino" 2010 "Vancouver" 2014 "Sochi"}))) 
 
;10. Functions can be defined inline: 
    (= __ ((fn [n] (* __ n)) 2)) 
    (= 20 ((fn [n] (* 10 n ))2)) 
 
;11. One function can beget another: 
    (= __ ((fn []((fn [a b](__ a b)) 4 5)))) 
    (= 9 ((fn []((fn [a b](+ a b)) 4 5)))) 
 
;12. Higher­order functions take function arguments
    (= 25 ( ___ (fn [n](* n n)))) 
    (= 25 ((fn [o] (o 5) ) (fn [n] (* n n)))) 
 
;13. You will face many decisions
    
    (=__(if (false? (= 4 5)) :a :b)) 
    (= :a (if (false? (= 4 5)) :a :b)) 
 
;14. Your alternative may be interesting
    (= :glory (if (not (empty? ())) :doom __)) 
    (= :glory (if (not (empty? ())) :doom :glory)) 
 
;15. Iteration provides an infinite lazy sequence: 
    (= __ (take 20(iterate inc 0))) 
    (= [0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19] (take 20(iterate inc 0))) 
 
;16. Iteration can be used for repetition
    (= (repeat 100 :foo) (take 100(iterate ___ :foo)))) 
    (= (repeat 100 :foo) (take 100(iterate (fn [f] f) :foo))) 
 
;17. Sequence comprehensions can bind each element in turn to a symbol
    (= __ (for [index (range 6)] index)) 
    (= (take 6 (iterate inc 0)) (for[index(range 6)] index)) 
 
;18. Sequence comprehensions can easily emulate mapping

    (='(0 1 4 9 16 25) (map (fn [index] (* index index)) (range 6)) (for[index (range 6)] __)) 
    (= '(0 1 4 9 16 25)
    (map (fn [index](* index index))
    (range 6))
    (for [index (range 6)]
    (* index index)
    ))
 
;19. Find out whether a list is a palindrome. A palindrome can be read forward or backward; e.g. (xamax).
    (= '(0 1 4 9 16 25)
    (map (fn [index](* index index))
    (range 6))
    (for [index (range 6)]
    (* index index)
    ))
 
;20. Use recursion to find the number of elements in a list.
    (def mylist '(1 3 4 3 1))
    (= mylist (into () (for [n mylist] n)))
    
{% endhighlight %}
    
## Advanced Koans

{% highlight clojure %}

;1.  Sorting a list of lists according to length of sublists.
;We suppose that a list contains elements that are lists themselves. The objective is to sort the elements of this list according to their length. E.g. short lists first, longer lists later, or vice versa. For example:
    (lsort '((a b c)(d e)(f g h)(d e)(i j k l)(m n)(o))) 
    ((O)(D E)(D E)(M N)(A B C)(F G H)(I J K L)) 
    (defn lsort [list] (sort-by count list)) 

;2. Sorting a list of lists according to frequency of the lengths of sublists
;Again, we suppose that a list contains elements that are lists themselves. But this time the objective is to sort the elements of this list according to their length frequency; i.e., in the default, where sorting is done ascendingly, lists with rare lengths are placed first, others with a more frequent length come later. For example:
    (lfsort'((abc)(de)(fgh)(de)(ijkl)(mn)(o))) ((ijkl)(o)(abc)(fgh)(de)(de)(mn)) 
 
;Note that in this example, the first two lists in the result have length 4 and 1, both lengths appear just once. The third and fourth lists have length 3 which appears twice (there are two list of this length). And finally, the last three lists have length 2: the most frequent length.
   (defn cleanUp [source](mapcat  #(if (sequential? %) % [%]) (sort-by count (vals (group-by count source)))))
 
;3.  English number words.
;On financial documents, like cheques, numbers must sometimes be written in full words. Example: 175 must be written as one­-seven-­five. Write a function called as-words to print (non­ negative) integer numbers in full words.
    (defn as-words [n]
        (let [words '("zero" "one" "two" "three" "four" "five" "six" "seven" "eight" "nine")](clojure.string/join "-" (map (fn [x] (nth words (Integer. (re-find #"\d" (str x)) ))) (str n)))))
    
;4.  Goldbach's conjecture.
;Goldbach's conjecture says that every positive even number greater than 2 is the sum of two prime numbers.
;Example: 28 = 5 + 23. It is one of the most famous facts in number theory that has not been proved to be correct in the general case. It has been numerically confirmed up to very large numbers. Write a function to find the two prime numbers that sum up to a given even integer. For example:
    (goldbach 28) 
    5 23
    (defn isPrime? [n]
        (if (= n 2)
        true
        (not(reduce #(or %1 %2) (for [i (range 2 n)] (= 0 (mod n i)))))
        )
    )
    (defn goldbach [n]
        (loop [i 2]
            (if (and (isPrime? i) (isPrime? (- n i)))
                (str i " " (- n i))
                (recur (+ i 1))
            )
        )
    )

{% endhighlight %}
