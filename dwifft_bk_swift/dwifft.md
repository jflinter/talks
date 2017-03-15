# Dwifft!

---

# 2013

^ Only iOS developer at Grouper (an online dating site)
^ Just finished our migration to iOS 7
^ I am in over my head
^ I had to read a bunch of my code from 3 years ago as part of writing this talk, so take pity on me

---

![inline](images/grouper.png)

^ I was trying to make a chat feature, trying to make it really good

---

- `UITableView`
- `DAKeyboardControl`
- Core Data?

^ Ok, so what did it take to build a screen like that?

---

```swift
tableViwe.reloadData() // amateur hour
```

^ you want animated changes.

---

```swift
// UITableView

func insertRows(
  at indexPaths: [IndexPath],
  with animation: UITableViewRowAnimation
)
```

^ But how do you figure out the index paths?

---

```swift
// NSFetchedResultsControllerDelegate

func controller(
  _ controller: NSFetchedResultsController,
  didChange anObject: Any,
  at indexPath: IndexPath?,
  for type: NSFetchedResultsChangeType,
  newIndexPath: IndexPath?
)
```

^ Turns out this function is perfectly matched up with the UITableView APIs! No problem, I thought - I'll just spin up a core data stack, dump new messages in there, and then retrieve them with my fetched results controller! (It's worth noting, this was my first time using Core Data.)

---

## "Now you have two problems"

^ So, a few thousand crashes later, I ended up pulling this code from the App Store in an emergency update, and just going back to calling reloadData when new messages came in. I wish I knew what the crash was today, but at the time it was basically "horrifying core data stacktrace", and I didn't have time to figure it out, and of course it was no big deal and none of this was worth the energy in the first place.

---

# 2015
^ Fast forward to 2015. I'm on a plane from new york to SF, visiting my new job. I've been pretending that I know how to write Swift for about a year, and it's becoming increasingly difficult, so I decide that I'm just going to spend the flight looking up random algorithms on the internet and trying to implement them in a Swift playground. I know how to party when I fly.

---

## Longest Common Subsequence

## SM**AR**T**P**H**ON**E
## H**ARPO**O**N**

^ So one of the algorithms I picked was called the Longest Common Subsequence problem. The problem states, given two sequences of values, find the longest not-necessarily-contiguous subsequence that they both share. So, for example, if our two inputs are SMARTPHONE and HARPOON, the longest common subsequence is ARPON. I spent way too long trying to come up with a decent word pair to use here.

---

$$
LCS\left(X_{i},Y_{j}\right) =
\begin{cases}
Ø
& \mbox{ if }\ i = 0 \mbox{ or }  j = 0 \\
  \textrm{  } LCS\left(X_{i-1},Y_{j-1}\right) \frown x_{i}
& \mbox{ if } x_i = y_j \\
  \mbox{longest}\left(LCS\left(X_{i},Y_{j-1}\right),LCS\left(X_{i-1},Y_{j}\right)\right)
& \mbox{ if } x_i \ne y_j \\
\end{cases}
$$

<br>
`// I stole this from Wikipedia`

^ So I have this ridiculous LaTEX formula that I stole from Wikipedia that actually isn't that bad when you break it down. LCS is a simple recursive function. You start at the back of both words, and advance two pointers backwards. If both pointers are pointing to the same letter, you add that letter to your LCS. If not, you try deleting the last letter of each word, call LCS again, and take the longer of the two.

---

# LCS -> Diff

### (-S) (-M) **A R** (-T) **P** (-H) **O N** (-E) -> ARPON
### (+H) **A R P O** (+O) **N** -> HARPOON

^ So, the interesting thing about LCS, that I started to realize on the plane, is that once you have the LCS between two sequences, you've also calculated the minimum series of transformations needed to turn one list into the other. You simply delete all the things from the first sequence that aren't in the LCS, then add all the things from the second word that aren't in it to the LCS, and you have a series of edit transformations.

---

# Dwifft!
## (Swift Diff)

---
```swift
enum DiffStep<T: Equatable> {
  case insert(Int, T)
  case delete(Int, T)
}

extension Array where Element: Equatable {
  func diff(other: [Element]) -> [DiffStep<Element>] {}
}
```

---

```swift
class TableViewDiffCalculator<T: Equatable> {
  let tableView: UITableView
  var rows: [T] // calls tableView updates
}
```

---

![inline](images/stuff.gif)
## Now what?
### (Alternate slide title: Dwifft diff)

---

# Vision
## (Dwission? Ok, I'll stop)
## https://www.stilldrinking.org/programming-sucks

^ I want to read you a passage from one of my favorite essays, titled "programming sucks."

^ Every programmer occasionally, when nobody's home, turns off the lights, pours a glass of scotch, puts on some light German electronica, and opens up a file on their computer[...] They read over the lines, and weep at their beauty, then the tears turn bitter as they remember the rest of the files and the inevitable collapse of all that is good and true in the world.

^ This file is Good Code. It has sensible and consistent names for functions and variables. It's concise. It doesn't do anything obviously stupid. It has never had to live in the wild, or answer to a sales team. It does exactly one, mundane, specific thing, and it does it well. It was written by a single person, and never touched by another. It reads like poetry written by someone over thirty.

^ I want Dwifft to be this for me.

---

## In practice

- simplest possible implementation (unix philosophy)
- 𝚫 -> 0
- close most PRs
- lots of tests
- minimize dependencies

---

# Announcing Dwifft v0.6!

```swift
class TableViewDiffCalculator<S: Equatable, T: Equatable> {
  var rows: [T] // deprecated!
  var sectionsAndRows: [(S, [T])] // THE FUTURE
}
```

^ Yes, someday you'll tell your grandkids, I was there when Dwifft 0.6 was announced.
^ Instead of just an array of rows, you now pass an array of tuples. But the magical thing is that now Dwifft will basically do a 2-dimensional diff on your array, and figure out the necessary section changes. You might be wondering, what's with that weird array of tuples thing. It's kind of awkward to write, but all it is is an ordered dictionary. This was originally just an array of arrays of T, but that had to change because sections themselves have meaning.
^ This was a really interesting change - I had this massively complicated strategy for doing this that I was sort of planning for like a year. I was going to change the diff algorithm into a tree diffing algorithm, then write code to effectively transform a 2D array into a tree, so that we could do diffs of arbitrary depth. But that was going to be really hard. And then my friend Jeremy idly suggested over breakfast, why don't you just treat the section boundaries themselves like array elements? So, just flatten your 2D arrays of sections and rows into 1D arrays that contain some objects, and some "section placeholders". Then diff those 1D arrays. That, as it turns out, is totally doable, and how this works.

---

# To 1.0

- ~~`sectionsAndRows`~~
- Coalesce delete/insert operations into moves
- macOS + tvOS targets
- Quick tests
- Swift ABI stability (ha ha)

---

