---
layout: default
---
# Carrotland Solution

Strategy: make a grid of integer points larger than the triangle, and test
each for inclusion with a point-in-triangle test.
http://stackoverflow.com/questions/2049582/how-to-determine-a-point-
in-a-triangle

That wasn't fast enough, then I did horizontal scan lines, figuring
out the leftmost and rightmost integer points inside the triangle, and
interpolating.

Then I did more reading and found Pick's theorem, at which point it gets
easy.


    import fractions
    def answer(vertices):
        v1, v2, v3 = vertices
        x1, y1 = v1
        x2, y2 = v2
        x3, y3 = v3
        area = 0.5 * abs(x1*(y2-y3) + x2*(y3-y1) + x3*(y1-y2))

        # http://math.stackexchange.com/questions/628117/how-to-count-lattice-points-on-a-line
        def count_lattice_points_on_line(v1, v2):
            return fractions.gcd(abs(v1[0]-v2[0]), abs(v1[1]-v2[1])) + 1

        boundary_points = (
            count_lattice_points_on_line(v1, v2) +
            count_lattice_points_on_line(v1, v3) +
            count_lattice_points_on_line(v2, v3) -
            3) # because each endpoint was counted twice

        # Pick's theorem: Area = N(interior lattice points) +
        #                         0.5(N(boundary lattice points)) - 1

        interior_points = int(area - 0.5*(boundary_points) + 1)
        return interior_points



[Back to Foobar challenges]({{ "/2015/08/01/google-foobar.html" | prepend: site.baseurl }})

