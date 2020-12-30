---
layout: default
permalink: /assignment/clojure-pathfinding/
---
# Clojure Pathfinding
## Assignment Criteria

*   Apply the functional style of programming to a range of simple problems
*   Implement a complex application in a functional language
*   Evaluate the merits of functional programming

{% highlight clojure %}
(def STARTROOM 8)
(def number_of_nodes 50)
(def distances (make-array Integer (+ number_of_nodes 1)))
(def daParents [])
(def settled ())
(def queue ())
(def tree {})
(def robot{:name "robotBasic" :currentRoom STARTROOM :Package false :Destination nil})
(def robotAdv{:name "robotAdvanced" :currentRoom STARTROOM :Packages [{:Destination 12}{:Destination 15}]})

(defn reset[]
    (def distances (make-array Integer (+ number_of_nodes 1)))
    (def daParents [])
    (def settled ())
    (def queue ())
    (def tree {})
)

(def adjacencyMatrix[
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,2,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,1,1,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,1,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,1,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,1,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,1,0,0,0,0,0,0,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,1,0,0,0,0,0,0,0,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,2,0,0]
	[0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,1,0,0,0,0,0,0,0,0,2]
	[0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,1,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,1,1,1,0,2,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,1,1,0,0,2,0,0,0]
	[0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,1,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,1,0,0,0,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,1,0,1,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,1,1,0,0,0,0]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,1]
	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,1,1,0]])

(def a (to-array-2d adjacencyMatrix))

(defn makeTree[source]
    (doseq [i (range 1 (+ 1 number_of_nodes))]
        (doseq [j (range 1 (+ 1 number_of_nodes))]
            (if (and (not (= (aget a i j) 0)) (not (= (aget a i j) Integer/MAX_VALUE)))
                (if (= (aget distances i) (+ (aget a i j) (aget distances j)))
                    (def daParents(conj daParents {:a j :b i}))
                    ()
                )
                ()
            )
        )
    )
    (def daParents (sort-by :a daParents))
)

(defn existsInTree?[tree child]
    (if(not(empty? (filter #(or (= % (:a child)) (= % (:b child))) (for [x (:children tree)] (:node x)) )))
    true
    false
    )
)

(defn hasChildren[nodeIn]
    (not (= [] (:children nodeIn)))
)

(defn addChild[t child]
    (assoc-in t 
        [:children] 
        (conj (:children t) child)
    )
)

(defn makeNode[value children]
    	{:node value :children (into [] children)}
)

(defn addToTree[treeIn child]
    (if (or (= (:node treeIn) (:a child)) (= (:node treeIn) (:b child)))
        (if (= (:a child) (:node treeIn))
            (addChild treeIn (makeNode (:b child) nil))
            (addChild treeIn (makeNode (:a child) nil))
        )
        (if (existsInTree? treeIn child)
            {:node (:node treeIn) :children 
                (for [x (:children treeIn)] 
                    (if (= (:node x) (:a child))
                        (addChild x (makeNode (:b child) nil))
                        (if (= (:node x) (:b child))
                            (addChild x (makeNode (:a child) nil))
                            x
                        )
                    )
                )
             }
            {:node (:node treeIn) :children 
                (for [x (:children treeIn)] 
                    (if (hasChildren x)
                        (addToTree x child)
                        x
                    )
                )
            }

        )
    )
)

(defn printTree[source]
    (def temp (first (filter #(= (:a %) source) daParents)))
    (def daParents (filter #(not(= % temp)) daParents)
)

;The 2 println statements below are needed as without them an error occours where one of the nodes is duplicated, not quite sure why this happens though
(println daParents)
(println)
(def tree {:node (:a temp) :children [{:node (:b temp) :children []}]})
(while (not (empty? daParents))
    (doseq[p daParents]
        (def oldTree tree)
        (def tree (addToTree tree p))
        (if (not (= oldTree tree))
            (def daParents (filter #(not (= % p)) daParents))
            ()
        )
    )
)
tree
)

;The following functions are used to generate the minimum distances
(defn getNodeWithMinimumDistanceFromQueue[]
    (def nodee (first queue))
    (def minimum (aget distances (first queue)))
    (def distTemp (seq distances))
    (doseq [item (range 1 (+ 2 number_of_nodes))]
        (def currentDist (first distTemp))
        (if (some #{currentDist} queue)
            (if (<=(aget distances currentDist) minimum)
                (
                (def minimum (aget distances currentDist))
                (def nodee currentDist)
                )
                ()
            )
            ()
        )
    )
    (def queue (rest queue))
    nodee
)

(defn doThings[evaluationNode destinationNode]
    (def edgeDistance -1)
    (def newDistance -1)
    (def edgeDistance (aget a evaluationNode destinationNode))
    (def newDistance (+ (aget distances evaluationNode) edgeDistance))
    (if (< newDistance (aget distances destinationNode))
        (aset distances destinationNode (int newDistance))
        (def newDistance (aget distances destinationNode))
    )
    (def queue (concat queue (list destinationNode)))
)
    
(defn evaluateNeighbours[evaluationNode]
    (doseq [item (range 1 (+ 1 number_of_nodes))]		
        (if(not(some #{item} settled))
            (if (not (= (aget a evaluationNode item) Integer/MAX_VALUE))
                (doThings evaluationNode item)
                ()
            )
            ()
        )
    )   
)

;This runs the dijkstra algorithm, once the dijkstra algorithm has finished the tree is the generated via the function makeTree
(defn dijkstraAlgorithm[source]
    (reset)
    (def daTree [:node source :cost 0])
    (doseq [x (range 1 (+ 1 number_of_nodes))] (aset distances x Integer/MAX_VALUE))
    (doseq [x (range 1 (+ 1 number_of_nodes))] 
    (doseq [y (range 1 (+ 1 number_of_nodes))]
    (if (= 0 (aget a x y))
        (aset a x y Integer/MAX_VALUE)
        ()
    )
    )
)

(def queue (seq (list source)))
    (aset distances source (int 0))
    (loop [q queue]
        (when (not-empty q)
            (def evaluationNode (getNodeWithMinimumDistanceFromQueue))
            (def settled (conj settled evaluationNode))
            (evaluateNeighbours evaluationNode)
            (recur queue)
        )
    )
    (makeTree source)
)

;These funstions are used to calculate the shortest/ideal/best path for navigaating several rooms
(defn AToB[a b]
    (dijkstraAlgorithm a)
    (println (str a " to " b " has cost: " (nth (seq distances) b)))
    (nth (seq distances) b)
)

(defn pathToA[a]
    (def results {:a (AToB STARTROOM a)})
    
    (println (sort-by last results))
    (dijkstraAlgorithm a)
    (println (printTree a))
)

(defn pathToAB[a b]
    (def results {:ab (+ (AToB STARTROOM a) (AToB a b))
                  :ba (+ (AToB STARTROOM b) (AToB b a))})
                  
    (println (sort-by last results))
    (dijkstraAlgorithm a)
    (println (printTree a))
    (dijkstraAlgorithm b)
    (println (printTree b))
)

(defn pathToABC[a b c]
    (def results {:abc (+ (AToB STARTROOM a) (AToB a b) (AToB b c)) 
                :acb (+ (AToB STARTROOM a) (AToB a c) (AToB c b)) 
                :bac (+ (AToB STARTROOM b) (AToB b a) (AToB a c))
                :bca (+ (AToB STARTROOM b) (AToB b c) (AToB c a))
                :cab (+ (AToB STARTROOM c) (AToB c a) (AToB a b)) 
                :cba (+ (AToB STARTROOM c) (AToB c b) (AToB b a))}
    )

    (println (sort-by last results))
    (dijkstraAlgorithm a)
    (println (printTree a))
    (dijkstraAlgorithm b)
    (println (printTree b))
    (dijkstraAlgorithm c)
    (println (printTree c))
)

;These functions are used for controlling and managing the robot
(defn displayRobot[r]
    (println (:name r) " is in room: " (:currentRoom r))
    (if (= true (:Package r))
        (println "Package for room " (:Destination r))
        (if (= nil (:Package r))
            (if (or (= [nil] (:Packages r)) (= [] (:Packages r)))
                (println "No Packages")
                (println "Packages: " (:Packages r))
            )
            (println "No Packages")
        )
    )
)

(defn makePackage[location]
    {:Destination location}
)

(defn addPackage[r package]
    (if (= (:name r) "robotBasic")
        {:name (:name r) :currentRoom (:currentRoom r) :Package true :Destination (:Destination package)}
        {:name (:name r) :currentRoom (:currentRoom r) :Packages (conj (:Packages r) package)}
    )
)

(defn moveRobot[r location]
    (if (= (:name r) "robotBasic")
        {:name (:name r) :currentRoom location :Package (:Package r) :Destination (:Destination r)}
        {:name (:name r) :currentRoom location :Packages (:Packages r)}
    )
)

(defn canTravel?[r location tree]
    ;find the robots current location in the tree, if the desired location is either the parent of the the node of a child of the node then the robot can travel otherwise the move is not valid
)



;(displayRobot robotAdv)
(def robotAdv (addPackage robotAdv (makePackage 10)))
(def robotAdv (moveRobot robotAdv 12))
;(displayRobot robotAdv)

;(displayRobot robot)
(def robot (addPackage robot (makePackage 10)))
(def robot (moveRobot robot 12))
;(displayRobot robot)

;the following numbers do not work for tree generation however minimum distances can be calculated for them 14 15 21 30 31 37 45 50 

;(def source 7)
;(dijkstraAlgorithm source)
;(println (printTree source))

;(pathToA 16)
;(pathToAB 27 36)
(pathToABC 27 36 16)

{% endhighlight %}
