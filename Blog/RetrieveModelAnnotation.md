# Retrieve model annotation 2.0

On my job, I came across a VBa program that consisted of 2 parts. The second part would add dimensions to your drawing. The first part would let you select faces/edges and add properties to it. Pairs of faces/edges are used to create dimensions on a drawing. Those properties included things like the position of the dimension text and to which view the dimension belongs.

This program came across my desk because no one knew how it worked and it stopped working when we migrated to a new version of Inventor. I usually have an opinion on how to improve things. And that was also here the case. In my opinion, there were 3 problems:

- Although there was a user interface it was not user-friendly. There was no way of knowing which face where used for a dimension.
- It was written in VBa and we should not be using VBa any more. ([Is VBA in Inventor obsolete?](./VbaIsObsolete.md))
- It's not very flexible.


