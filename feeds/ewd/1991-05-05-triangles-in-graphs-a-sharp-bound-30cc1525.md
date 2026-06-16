---
title: "Triangles in graphs: a sharp bound (EWD1101)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1101.html
published: "1991-05-05T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1101.PDF
---

# Triangles in graphs: a sharp bound

Triangles in graphs; a sharp bound

We consider undirected graphs without autoloops and without multiple edges—i.e. each edge connects a pair of different vertices, and a pair of vertices is connected by at most 1 edge—. A triangle is a set of 3 vertices such that any two of them are connected by an edge of the graph; the graph is then said to "contain" that triangle.

Theorem     Let M.V be the maximum number of edges of a graph with V vertices such that the graph need not contain a triangle; then M.(2k) = k2 and M.(2k-1) = k∙(k-1)         .

Proof          Both equations for M have the form M.(a+b) = a∙b.  Showing that a graph with a+b vertices and a∙b edges need not contain a triangle is done by constructing a witness. Paint a of the vertices red and the remaining b vertices blue; introduce a∙b edges by connecting each red vertex with each blue vertex. Since in the resulting (bipartite) graph colours along a path alternate, cycles have even length and hence the graph contains no triangle.

That a graph with more edges —if feasible— contains a triangle is shown by mathematical inductions.

We confine our attention to graphs with at least 3 vertices, and for the base of the induction we consider (a, b) = (1, 2) and (a, b) = (2, 2).  In the first case, we have to show that a graph with #vertices = 1+2 and #edges > 1∙2 contains a triangle: such a graph has 3 vertices and 3 edges and is a triangle. In the second case, we have to show that a graph with #vertices = 2+2 and #edges > 2∙2 contains a triangle:  such a graph has at least 5 edges and hence contains the complete 4-graph with 1 edge missing, which contains even 2 triangles.

For the induction step we show for any a, b:

(any graph with #vertices = a+b and #edges > a∙b contains a triangle)      ⇒

(any graph with #vertices = (a+1)+(b+1) and #edges > (a+1)∙(b+1) contains a triangle).

To this end we consider a graph with (a+1)+(b+1)   —= a+b+2—   vertices and more than (a+1)∙(b+1)   —= a∙b+a+b+1—   edges and focus our attention on one of its edges e.  Now we consider two cases: e is or is not an edge of a triangle. In the former case the graph obviously contains a triangle. If e is not the edge of a triangle, however, it is connected by at most a+b edges to the subgraph spanned by the remaining a+b vertices; the total graph having more than  a∙b+a+b+1 edges, we conclude that in this case the subgraph of a+b vertices has more than a∙b edges and, according to the antecedent of the demonstrandum, contains a triangle. In view of the base we have demonstrated the presence of a triangle for a+1=b or a=b and b≥2.

(End of Proof).

The above proof is noteworthy for two reasons, firstly because even and odd number of vertices could share the same induction step, and secondly because the presence of a triangle in a graph with 4 vertices and 5 edges has been demonstrated without case analysis. (Take the experiment and ask your friends.)

Austin, 5 May 1991

prof.dr. Edsger W.Dijkstra,

Department of Computer Sciences,

The University of Texas at Austin,

Austin, TX 78712 – 1188

USA

Transcribed by Guy Haworth

Revised Sunday,
29-Dec-2013

.
