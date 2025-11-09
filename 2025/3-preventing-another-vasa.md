# Preventing Another Vasa

> Note: The examples in this editorial are written to take advantage of some newer Java features available in Java 17+.  The record classes can be written as traditional classes instead.

Oh boy, a math heavy problem (or if you're more adventurous, you can technically simulate this problem instead).  We will follow the math approach for this editorial.

Looking at this problem, it is technically just finding the intersections between different line segments and circles.  Each boat has a start position, a direction and speed.  From that, we can calculate the end point after two hours.  From there we can find all the intersections.  We simply need to calculate when each boat will arrive at the intersection and if it will arrive at the same time.

The islands are thankfully stationary so the only worry there is whether we have one intersection or two.

Ultimately for each boat, we stop checking that boat after its first crash.  The one major hijink being if there are two crashes simultaneously, we have to consider the island crash as the one we report.

Whew..., that's a lot to consider, so let's start breaking it down.

We'll be using Java for this problem.

## Computing Line segments

Let's start by creating a coordinate record.  This record will have a couple of helper methods just simply to help with the distance and intersection calculations.

```java
public record Coordinate(BigDecimal x, BigDecimal y) {
    Coordinate subtract(Coordinate other) {
        return new Coordinate(x.subtract(other.x), y.subtract(other.y));
    }

    Coordinate add(Coordinate other) {
        return new Coordinate(x.add(other.x), y.add(other.y));
    }

    Coordinate multiply(BigDecimal scalar) {
        return new Coordinate(x.multiply(scalar), y.multiply(scalar));
    }

    BigDecimal dot(Coordinate other) {
        return x.multiply(other.x).add(y.multiply(other.y));
    }

    BigDecimal magnitude() {
        return x.multiply(x).add(y.multiply(y)).sqrt(MathContext.DECIMAL64);
    }

    BigDecimal crossProduct(Coordinate other) {
        return x.multiply(other.y).subtract(y.multiply(other.x));
    }

    BigDecimal distanceTo(Coordinate other) {
        return subtract(other).magnitude();
    }
}
```

We'll also create a line record.

```java
public record Line(Coordinate start, Coordinate end) {}
```

From here, we can finally make a boat.

```java
private static final Integer PRECISION = 5;
private static final BigDecimal ANGLE_ADJUST = new BigDecimal(270);
private static final BigDecimal SIMULATION_MINUTES = new BigDecimal(120);
private static final BigDecimal DEGREES_TO_RADIANS = new BigDecimal("0.017453292519943295");

public static class Boat {
    private final Coordinate start;
    private Coordinate end;
    private final BigDecimal direction, speed;

    public Boat(int id, Coordinate start, BigDecimal direction, BigDecimal speed) {
        super(id);
        this.start = start;
        this.direction = direction;
        this.speed = speed;
        this.end = end();
    }

    public Coordinate start() {
        return start;
    }

    public Coordinate end() {
        if(end == null) {
            final BigDecimal endX = start.x.add(speed.multiply(SIMULATION_MINUTES).multiply(cos(toRadians(direction.add(ANGLE_ADJUST)))));
            final BigDecimal endY = start.y.subtract(speed.multiply(SIMULATION_MINUTES).multiply(sin(toRadians(direction.add(ANGLE_ADJUST)))));
            end = new Coordinate(endX.setScale(PRECISION, RoundingMode.HALF_UP), endY.setScale(PRECISION, RoundingMode.HALF_UP));
        }
        return end;
    }

    private BigDecimal sin(BigDecimal angle) {
        return BigDecimal.valueOf(Math.sin(angle.doubleValue()));
    }

    private BigDecimal cos(BigDecimal angle) {
        return BigDecimal.valueOf(Math.cos(angle.doubleValue()));
    }

    private BigDecimal toRadians(BigDecimal angle) {
        return angle.multiply(DEGREES_TO_RADIANS);
    }

    public Line line() {
        return new Line(start, end());
    }
}
```

And we can make an island as well.

```java
public static class Island extends CollidableObject {
    private final Coordinate center;
    private final BigDecimal radius;

    public Island(int id, Coordinate center, BigDecimal radius) {
        super(id);
        this.center = center;
        this.radius = radius;
    }
}
```

## Line / Line Intersections

We can use vectors to determine whether two lines intersect, but if we do this, we run into an interesting problem.  The simulation only runs for two hours, and the lines might intersect after that.  We definitely want to ignore those intersections.

```java
private static final Integer PRECISION = 5;
private static final BigDecimal EPSILON = new BigDecimal("1e-10");

public record Line(Coordinate start, Coordinate end) {
    public Coordinate intersection(Line other) {
        // Line 1 represented as a1x + b1y = c1
        BigDecimal a1 = end.y.subtract(start.y);
        BigDecimal b1 = start.x.subtract(end.x);
        BigDecimal c1 = a1.multiply(start.x).add(b1.multiply(start.y));

        // Line 2 represented as a2x + b2y = c2
        BigDecimal a2 = other.end.y.subtract(other.start.y);
        BigDecimal b2 = other.start.x.subtract(other.end.x);
        BigDecimal c2 = a2.multiply(other.start.x).add(b2.multiply(other.start.y));

        // Calculate determinant
        BigDecimal determinant = a1.multiply(b2).subtract(a2.multiply(b1));

        // If determinant is zero, lines are parallel or coincident
        if (determinant.abs().compareTo(EPSILON) <= 0) {
            return null;
        }

        // Calculate intersection point
        BigDecimal x = (b2.multiply(c1).subtract(b1.multiply(c2))).divide(determinant, PRECISION, RoundingMode.HALF_UP);
        BigDecimal y = (a1.multiply(c2).subtract(a2.multiply(c1))).divide(determinant, PRECISION, RoundingMode.HALF_UP);
        var intersection = new Coordinate(x.setScale(PRECISION, RoundingMode.HALF_UP), y.setScale(PRECISION, RoundingMode.HALF_UP));

        // Check if intersection point is within both line segments
        if (isPointOnLineSegment(this, intersection) && isPointOnLineSegment(other, intersection)) {
            return intersection;
        }

        // Intersection point exists but not within the line segments
        return null;
    }

    private boolean isPointOnLineSegment(Line line, Coordinate coordinate) {
        // Check if point (x, y) is on the line segment from lineStart to lineEnd
        BigDecimal minX = line.start.x.min(line.end.x);
        BigDecimal maxX = line.start.x.max(line.end.x);
        BigDecimal minY = line.start.y.min(line.end.y);
        BigDecimal maxY = line.start.y.max(line.end.y);

        return coordinate.x.compareTo(minX) >= 0 && coordinate.x.compareTo(maxX) <= 0 &&
                coordinate.y.compareTo(minY) >= 0 && coordinate.y.compareTo(maxY) <= 0;
    }
}
```

## Line / Circle Intersections

Likewise, we can use vectors to find our circle intersections.

```java
private static final Integer PRECISION = 5;
private static final BigDecimal NEG_ONE = new BigDecimal("-1");
private static final BigDecimal EPSILON = new BigDecimal("1e-10");
private static final BigDecimal TWO = new BigDecimal(2);
private static final BigDecimal FOUR = new BigDecimal(4);

public record Line(Coordinate start, Coordinate end) {
    public List<Coordinate> intersect(Island island) {
        List<Coordinate> intersections = new ArrayList<>();

        // Convert line to parametric form: P = start + t * direction
        BigDecimal dx = this.end.x.subtract(this.start.x);
        BigDecimal dy = this.end.y.subtract(this.start.y);

        // Vector from line start to circle center
        BigDecimal fx = this.start.x.subtract(island.center.x);
        BigDecimal fy = this.start.y.subtract(island.center.y);

        // Quadratic equation coefficients: at^2 + bt + c = 0
        BigDecimal a = dx.multiply(dx).add(dy.multiply(dy));
        BigDecimal b = TWO.multiply(fx.multiply(dx).add(fy.multiply(dy)));
        BigDecimal c = fx.multiply(fx).add(fy.multiply(fy)).subtract(island.radius.multiply(island.radius));

        BigDecimal discriminant = b.multiply(b).subtract(FOUR.multiply(a).multiply(c));

        if (discriminant.compareTo(BigDecimal.ZERO) < 0) {
            // No intersection
            return intersections;
        }

        if(a.compareTo(EPSILON) <= 0 && b.compareTo(EPSILON) <= 0) {
            return intersections;
        }

        if (discriminant.compareTo(EPSILON) <= 0) {
            // One intersection (tangent)
            BigDecimal t = NEG_ONE.multiply(b).divide(TWO.multiply(a), PRECISION, RoundingMode.HALF_UP);
            if (isValidPoint(t)) {
                BigDecimal x = this.start.x.add(t.multiply(dx));
                BigDecimal y = this.start.y.add(t.multiply(dy));
                intersections.add(new Coordinate(x.setScale(PRECISION, RoundingMode.HALF_UP), y.setScale(PRECISION, RoundingMode.HALF_UP)));
            }
        } else {
            // Two intersections
            BigDecimal sqrtDiscriminant = discriminant.sqrt(MathContext.DECIMAL64);
            BigDecimal t1 = (NEG_ONE.multiply(b).subtract(sqrtDiscriminant)).divide(TWO.multiply(a), PRECISION, RoundingMode.HALF_UP);
            BigDecimal t2 = (NEG_ONE.multiply(b).add(sqrtDiscriminant)).divide(TWO.multiply(a), PRECISION, RoundingMode.HALF_UP);

            if (isValidPoint(t1)) {
                BigDecimal x1 = this.start.x.add(t1.multiply(dx));
                BigDecimal y1 = this.start.y.add(t1.multiply(dy));
                intersections.add(new Coordinate(x1.setScale(PRECISION, RoundingMode.HALF_UP), y1.setScale(PRECISION, RoundingMode.HALF_UP)));
            }

            if (isValidPoint(t2)) {
                BigDecimal x2 = this.start.x.add(t2.multiply(dx));
                BigDecimal y2 = this.start.y.add(t2.multiply(dy));
                intersections.add(new Coordinate(x2.setScale(PRECISION, RoundingMode.HALF_UP), y2.setScale(PRECISION, RoundingMode.HALF_UP)));
            }
        }

        return intersections;
    }

    private static boolean isValidPoint(BigDecimal t) {
        return t.compareTo(BigDecimal.ZERO) >= 0 && t.compareTo(BigDecimal.ONE) <= 0;
    }
}
```

# Overtaking / Head On Collisions

Now you might be thinking, wow that handles everything, awesome, but you would be incorrect.  There are two special types of collisions that aren't covered by intersections, head on collisions and rear end collisions.  Technically those are overlapping line segments not intersecting line segments.  Regardless, they are a special case that we have to handle.

```java
private static final Integer PRECISION = 5;
private static final BigDecimal SIMULATION_MINUTES = new BigDecimal(120);
private static final BigDecimal NEG_ONE = new BigDecimal("-1");
private static final BigDecimal EPSILON = new BigDecimal("1e-10");

public static class Boat extends CollidableObject {
    private final Coordinate start;
    private Coordinate end;
    private final BigDecimal direction, speed;

    ...

    public BigDecimal timeTo(Coordinate coordinate) {
        return start.distanceTo(coordinate).divide(speed, PRECISION, RoundingMode.HALF_UP);
    }

    public BigDecimal doesIntersect(Boat otherBoat) {
        // find intersection point
        var intersectionPoint = this.line().intersection(otherBoat.line());
        if(intersectionPoint != null) {
            var time1At = this.timeTo(intersectionPoint);
            var time2At = otherBoat.timeTo(intersectionPoint);
            if(time1At.subtract(time2At).abs().compareTo(EPSILON) <= 0) {
                return time1At;
            }
        } else if(this.line().doesOverlap(otherBoat.line())) {
            // do the boats overlap?
            if(this.direction.equals(otherBoat.direction)) {
                return calculateOvertakingTime(otherBoat);
            } else {
                return calculateHeadOnCollisionTime(otherBoat);
            }
        }
        return NEG_ONE;
    }

    private BigDecimal calculateOvertakingTime(Boat otherBoat) {
        // For overlapping boats, check if one is faster and will overtake
        if(this.start == otherBoat.start) {
            return BigDecimal.ZERO;
        }

        if(this.speed.compareTo(otherBoat.speed) == 0) {
            // Same speed, parallel movement - collision at start
            return NEG_ONE;
        }

        // Calculate relative speed and initial separation
        BigDecimal relativeSpeed = this.speed.subtract(otherBoat.speed).abs();
        BigDecimal initialSeparation = this.start.distanceTo(otherBoat.start);

        // Time for faster boat to catch up
        BigDecimal overtakeTime = initialSeparation.divide(relativeSpeed, PRECISION, RoundingMode.HALF_UP);

        // Check if overtaking happens within simulation time
        if(overtakeTime.compareTo(SIMULATION_MINUTES) <= 0) {
            return overtakeTime;
        }

        return NEG_ONE;
    }

    private BigDecimal calculateHeadOnCollisionTime(Boat otherBoat) {
        BigDecimal combinedSpeed = this.speed.add(otherBoat.speed);
        BigDecimal initialSeparation = this.start.distanceTo(otherBoat.start);

        BigDecimal collisionTime = initialSeparation.divide(combinedSpeed, PRECISION, RoundingMode.HALF_UP);

        if(collisionTime.compareTo(SIMULATION_MINUTES) <= 0) {
            return collisionTime;
        }

        return NEG_ONE;
    }

    public BigDecimal doesIntersect(Island island) {
        // find intersection points
        List<Coordinate> intersectionPoints = this.line().intersect(island);
        if(!intersectionPoints.isEmpty()) {
            BigDecimal min = null;
            for(var point : intersectionPoints) {
                BigDecimal time = this.timeTo(point);
                if(min == null || time.compareTo(min) < 0) {
                    min = time;
                }
            }
            return min;
        }
        return NEG_ONE;
    }
}
```

Whew...  That should cover every single instance.  Now, it should be possible to get every single collision possible.

```java
public record Collision(CollidableObject obj1, CollidableObject obj2) {}

TreeMap<BigDecimal, List<Collision>> collisions = new TreeMap<>();
for(Boat boat : boats) {
    for(Boat otherBoat : boats) {
        if(boat != otherBoat) {
            BigDecimal collisionTime = boat.doesIntersect(otherBoat);
            if(collisionTime.compareTo(BigDecimal.ZERO) >= 0) {
                if(!collisions.containsKey(collisionTime)) {
                    collisions.put(collisionTime, new ArrayList<>());
                }
                collisions.get(collisionTime).add(new Collision(boat, otherBoat));
            }
        }
    }
    for(Island island : islands) {
        BigDecimal collisionTime = boat.doesIntersect(island);
        if(collisionTime.compareTo(BigDecimal.ZERO) >= 0) {
            if(!collisions.containsKey(collisionTime)) {
                collisions.put(collisionTime, new ArrayList<>());
            }
            collisions.get(collisionTime).add(new Collision(boat, island));
        }
    }
}
```

TreeMaps in Java are lovely structures that automatically order their values based on the key, so after we do all the inserts, we can simply scan across the bottom.

```java
Map<Integer, String> results = new HashMap<>();
Set<Integer> alreadyCrashed = new HashSet<>();
for(var entries : collisions.entrySet()) {
    for(var collision : entries.getValue()) {
        // the boat must be uncrashed
        if(collision.obj2() instanceof Island && !alreadyCrashed.contains(collision.obj1().id())) {
            alreadyCrashed.add(collision.obj1().id());
            results.put(collision.obj1().id(), "Island");
        }
        // both boats have to be in an uncrashed state
        if(collision.obj2() instanceof Boat && !alreadyCrashed.contains(collision.obj1().id()) && !alreadyCrashed.contains(collision.obj2().id())) {
            alreadyCrashed.add(collision.obj1().id());
            alreadyCrashed.add(collision.obj2().id());
            results.put(collision.obj1().id(), "Boat");
            results.put(collision.obj2().id(), "Boat");
        }
    }
}

for(var boat : boats) {
    System.out.println(results.getOrDefault(boat.id(), "Safe"));
}
```

# Solution

```java
import java.math.BigDecimal;
import java.math.MathContext;
import java.math.RoundingMode;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Scanner;
import java.util.Set;
import java.util.TreeMap;

/* This file provides a solution for "Preventing Another Vasa", a programming contest problem
 * for the 2026 CCSC Programming Contest in Johnson City, TN.
 *
 * The problem's idea is rather straight-forward, you have boats and islands, and you wish to
 * detect if a collision is going to occur.  The difficulty of the problem stems from collisions
 * can occur both directly and indirectly.
 *
 * Direct Collisions:
 * - two line segments (boats) intersect at the same time frame
 * - a line segment (boat) and a circle (island) intersect
 * - a boat overtakes another boat traveling the same direction
 *
 * Each of the above situations is tested in the sample test cases and are handled slightly differently.
 */
public class Main {
    private static final Integer PRECISION = 5;
    private static final BigDecimal ANGLE_ADJUST = new BigDecimal(270);
    private static final BigDecimal SIMULATION_MINUTES = new BigDecimal(120);
    private static final BigDecimal DEGREES_TO_RADIANS = new BigDecimal("0.017453292519943295");
    private static final BigDecimal NEG_ONE = new BigDecimal("-1");
    private static final BigDecimal EPSILON = new BigDecimal("1e-10");
    private static final BigDecimal TWO = new BigDecimal(2);
    private static final BigDecimal FOUR = new BigDecimal(4);

    public record Coordinate(BigDecimal x, BigDecimal y) {
        /* helper methods for intersection and distance calculations */
        Coordinate subtract(Coordinate other) {
            return new Coordinate(x.subtract(other.x), y.subtract(other.y));
        }

        Coordinate add(Coordinate other) {
            return new Coordinate(x.add(other.x), y.add(other.y));
        }

        Coordinate multiply(BigDecimal scalar) {
            return new Coordinate(x.multiply(scalar), y.multiply(scalar));
        }

        BigDecimal dot(Coordinate other) {
            return x.multiply(other.x).add(y.multiply(other.y));
        }

        BigDecimal magnitude() {
            return x.multiply(x).add(y.multiply(y)).sqrt(MathContext.DECIMAL64);
        }

        BigDecimal crossProduct(Coordinate other) {
            return x.multiply(other.y).subtract(y.multiply(other.x));
        }

        BigDecimal distanceTo(Coordinate other) {
            return subtract(other).magnitude();
        }
    }

    public record Line(Coordinate start, Coordinate end) {
        /* do two line segments (boats) overlap */
        public boolean doesOverlap(Line other) {
            Coordinate v1 = end.subtract(start);
            Coordinate v2 = other.start.subtract(start);
            Coordinate v3 = other.end.subtract(start);

            boolean areColinear = v1.crossProduct(v2).abs().compareTo(EPSILON) <= 0 &&
                                  v1.crossProduct(v3).abs().compareTo(EPSILON) <= 0;

            if (!areColinear) {
                return false;
            }

            boolean useXAxis = end.x.subtract(start.x).abs().compareTo(end.y.subtract(start.y).abs()) >= 0;

            BigDecimal thisStart, thisEnd, otherStart, otherEnd;

            if (useXAxis) {
                thisStart = start.x;
                thisEnd = end.x;
                otherStart = other.start.x;
                otherEnd = other.end.x;
            } else {
                thisStart = start.y;
                thisEnd = end.y;
                otherStart = other.start.y;
                otherEnd = other.end.y;
            }

            // Ensure start <= end for both segments
            if (thisStart.compareTo(thisEnd) > 0) {
                BigDecimal temp = thisStart;
                thisStart = thisEnd;
                thisEnd = temp;
            }

            if (otherStart.compareTo(otherEnd) > 0) {
                BigDecimal temp = otherStart;
                otherStart = otherEnd;
                otherEnd = temp;
            }

            // Check if segments overlap: max(start1, start2) <= min(end1, end2)
            BigDecimal overlapStart = thisStart.max(otherStart);
            BigDecimal overlapEnd = thisEnd.min(otherEnd);

            return overlapStart.compareTo(overlapEnd) <= 0;
        }

        /* do two line segments (boats) intersect */
        public Coordinate intersection(Line other) {
            // Line 1 represented as a1x + b1y = c1
            BigDecimal a1 = end.y.subtract(start.y);
            BigDecimal b1 = start.x.subtract(end.x);
            BigDecimal c1 = a1.multiply(start.x).add(b1.multiply(start.y));

            // Line 2 represented as a2x + b2y = c2
            BigDecimal a2 = other.end.y.subtract(other.start.y);
            BigDecimal b2 = other.start.x.subtract(other.end.x);
            BigDecimal c2 = a2.multiply(other.start.x).add(b2.multiply(other.start.y));

            // Calculate determinant
            BigDecimal determinant = a1.multiply(b2).subtract(a2.multiply(b1));

            // If determinant is zero, lines are parallel or coincident
            if (determinant.abs().compareTo(EPSILON) <= 0) {
                return null;
            }

            // Calculate intersection point
            BigDecimal x = (b2.multiply(c1).subtract(b1.multiply(c2))).divide(determinant, PRECISION, RoundingMode.HALF_UP);
            BigDecimal y = (a1.multiply(c2).subtract(a2.multiply(c1))).divide(determinant, PRECISION, RoundingMode.HALF_UP);
            var intersection = new Coordinate(x.setScale(PRECISION, RoundingMode.HALF_UP), y.setScale(PRECISION, RoundingMode.HALF_UP));

            // Check if intersection point is within both line segments
            if (isPointOnLineSegment(this, intersection) && isPointOnLineSegment(other, intersection)) {
                return intersection;
            }

            // Intersection point exists but not within the line segments
            return null;
        }

        private boolean isPointOnLineSegment(Line line, Coordinate coordinate) {
            // Check if point (x, y) is on the line segment from lineStart to lineEnd
            BigDecimal minX = line.start.x.min(line.end.x);
            BigDecimal maxX = line.start.x.max(line.end.x);
            BigDecimal minY = line.start.y.min(line.end.y);
            BigDecimal maxY = line.start.y.max(line.end.y);

            return coordinate.x.compareTo(minX) >= 0 && coordinate.x.compareTo(maxX) <= 0 &&
                   coordinate.y.compareTo(minY) >= 0 && coordinate.y.compareTo(maxY) <= 0;
        }

        /* do a line segment (boat) and circle intersect */
        public List<Coordinate> intersect(Island island) {
            List<Coordinate> intersections = new ArrayList<>();

            // Convert line to parametric form: P = start + t * direction
            BigDecimal dx = this.end.x.subtract(this.start.x);
            BigDecimal dy = this.end.y.subtract(this.start.y);

            // Vector from line start to circle center
            BigDecimal fx = this.start.x.subtract(island.center.x);
            BigDecimal fy = this.start.y.subtract(island.center.y);

            // Quadratic equation coefficients: at^2 + bt + c = 0
            BigDecimal a = dx.multiply(dx).add(dy.multiply(dy));
            BigDecimal b = TWO.multiply(fx.multiply(dx).add(fy.multiply(dy)));
            BigDecimal c = fx.multiply(fx).add(fy.multiply(fy)).subtract(island.radius.multiply(island.radius));

            BigDecimal discriminant = b.multiply(b).subtract(FOUR.multiply(a).multiply(c));

            if (discriminant.compareTo(BigDecimal.ZERO) < 0) {
                // No intersection
                return intersections;
            }

            if(a.compareTo(EPSILON) <= 0 && b.compareTo(EPSILON) <= 0) {
                return intersections;
            }

            if (discriminant.compareTo(EPSILON) <= 0) {
                // One intersection (tangent)
                BigDecimal t = NEG_ONE.multiply(b).divide(TWO.multiply(a), PRECISION, RoundingMode.HALF_UP);
                if (isValidPoint(t)) {
                    BigDecimal x = this.start.x.add(t.multiply(dx));
                    BigDecimal y = this.start.y.add(t.multiply(dy));
                    intersections.add(new Coordinate(x.setScale(PRECISION, RoundingMode.HALF_UP), y.setScale(PRECISION, RoundingMode.HALF_UP)));
                }
            } else {
                // Two intersections
                BigDecimal sqrtDiscriminant = discriminant.sqrt(MathContext.DECIMAL64);
                BigDecimal t1 = (NEG_ONE.multiply(b).subtract(sqrtDiscriminant)).divide(TWO.multiply(a), PRECISION, RoundingMode.HALF_UP);
                BigDecimal t2 = (NEG_ONE.multiply(b).add(sqrtDiscriminant)).divide(TWO.multiply(a), PRECISION, RoundingMode.HALF_UP);

                if (isValidPoint(t1)) {
                    BigDecimal x1 = this.start.x.add(t1.multiply(dx));
                    BigDecimal y1 = this.start.y.add(t1.multiply(dy));
                    intersections.add(new Coordinate(x1.setScale(PRECISION, RoundingMode.HALF_UP), y1.setScale(PRECISION, RoundingMode.HALF_UP)));
                }

                if (isValidPoint(t2)) {
                    BigDecimal x2 = this.start.x.add(t2.multiply(dx));
                    BigDecimal y2 = this.start.y.add(t2.multiply(dy));
                    intersections.add(new Coordinate(x2.setScale(PRECISION, RoundingMode.HALF_UP), y2.setScale(PRECISION, RoundingMode.HALF_UP)));
                }
            }

            return intersections;
        }

        private static boolean isValidPoint(BigDecimal t) {
            return t.compareTo(BigDecimal.ZERO) >= 0 && t.compareTo(BigDecimal.ONE) <= 0;
        }
    }

    // helper class to allow boats and islands to be stored in same data type
    public static class CollidableObject {
        protected final int id;

        public CollidableObject(int id) {
            this.id = id;
        }

        public int id() {
            return id;
        }
    }

    public static class Boat extends CollidableObject {
        private final Coordinate start;
        private Coordinate end;
        private final BigDecimal direction, speed;

        public Boat(int id, Coordinate start, BigDecimal direction, BigDecimal speed) {
            super(id);
            this.start = start;
            this.direction = direction;
            this.speed = speed;
        }

        /* helper methods for calculating end point */
        private BigDecimal sin(BigDecimal angle) {
            return BigDecimal.valueOf(Math.sin(angle.doubleValue()));
        }

        private BigDecimal cos(BigDecimal angle) {
            return BigDecimal.valueOf(Math.cos(angle.doubleValue()));
        }

        private BigDecimal toRadians(BigDecimal angle) {
            return angle.multiply(DEGREES_TO_RADIANS);
        }

        /* Calculate end point after SIMULATION_MINUTES (and cache) */
        public Coordinate end() {
            if(end == null) {
                final BigDecimal endX = start.x.add(speed.multiply(SIMULATION_MINUTES).multiply(cos(toRadians(direction.add(ANGLE_ADJUST)))));
                final BigDecimal endY = start.y.subtract(speed.multiply(SIMULATION_MINUTES).multiply(sin(toRadians(direction.add(ANGLE_ADJUST)))));
                end = new Coordinate(endX.setScale(PRECISION, RoundingMode.HALF_UP), endY.setScale(PRECISION, RoundingMode.HALF_UP));
            }
            return end;
        }

        public Line line() {
            return new Line(start, end());
        }

        public BigDecimal timeTo(Coordinate coordinate) {
            return start.distanceTo(coordinate).divide(speed, PRECISION, RoundingMode.HALF_UP);
        }

        public BigDecimal doesIntersect(Boat otherBoat) {
            // find intersection point
            var intersectionPoint = this.line().intersection(otherBoat.line());
            if(intersectionPoint != null) {
                var time1At = this.timeTo(intersectionPoint);
                var time2At = otherBoat.timeTo(intersectionPoint);
                if(time1At.subtract(time2At).abs().compareTo(EPSILON) <= 0) {
                    return time1At;
                }
            } else if(this.line().doesOverlap(otherBoat.line())) {
                // do the boats overlap?
                if(this.direction.equals(otherBoat.direction)) {
                    return calculateOvertakingTime(otherBoat);
                } else {
                    return calculateHeadOnCollisionTime(otherBoat);
                }
            }
            return NEG_ONE;
        }

        private BigDecimal calculateOvertakingTime(Boat otherBoat) {
            // For overlapping boats, check if one is faster and will overtake
            if(this.start == otherBoat.start) {
                return BigDecimal.ZERO;
            }

            if(this.speed.compareTo(otherBoat.speed) == 0) {
                // Same speed, parallel movement - collision at start
                return NEG_ONE;
            }

            // Calculate relative speed and initial separation
            BigDecimal relativeSpeed = this.speed.subtract(otherBoat.speed).abs();
            BigDecimal initialSeparation = this.start.distanceTo(otherBoat.start);

            // Time for faster boat to catch up
            BigDecimal overtakeTime = initialSeparation.divide(relativeSpeed, PRECISION, RoundingMode.HALF_UP);

            // Check if overtaking happens within simulation time
            if(overtakeTime.compareTo(SIMULATION_MINUTES) <= 0) {
                return overtakeTime;
            }

            return NEG_ONE;
        }

        private BigDecimal calculateHeadOnCollisionTime(Boat otherBoat) {
            BigDecimal combinedSpeed = this.speed.add(otherBoat.speed);
            BigDecimal initialSeparation = this.start.distanceTo(otherBoat.start);

            BigDecimal collisionTime = initialSeparation.divide(combinedSpeed, PRECISION, RoundingMode.HALF_UP);

            if(collisionTime.compareTo(SIMULATION_MINUTES) <= 0) {
                return collisionTime;
            }

            return NEG_ONE;
        }

        public BigDecimal doesIntersect(Island island) {
            // find intersection points
            List<Coordinate> intersectionPoints = this.line().intersect(island);
            if(!intersectionPoints.isEmpty()) {
                BigDecimal min = null;
                for(var point : intersectionPoints) {
                    BigDecimal time = this.timeTo(point);
                    if(min == null || time.compareTo(min) < 0) {
                        min = time;
                    }
                }
                return min;
            }
            return NEG_ONE;
        }
    }

    public static class Island extends CollidableObject {
        private final Coordinate center;
        private final BigDecimal radius;

        public Island(int id, Coordinate center, BigDecimal radius) {
            super(id);
            this.center = center;
            this.radius = radius;
        }
    }

    public record Collision(CollidableObject obj1, CollidableObject obj2) {}

    public static void main(String[] args) {
        final List<Boat> boats = new ArrayList<>();
        final List<Island> islands = new ArrayList<>();

        /* Read In Data */
        Scanner scanner = new Scanner(System.in);
        int numberOfBoats = scanner.nextInt(), numberOfIslands = scanner.nextInt();
        int ids = 0;
        for(int i = 0; i < numberOfBoats; i++) {
            boats.add(new Boat(ids++, new Coordinate(scanner.nextBigDecimal(), scanner.nextBigDecimal()), scanner.nextBigDecimal(), scanner.nextBigDecimal()));
        }
        for(int i = 0; i < numberOfIslands; i++) {
            islands.add(new Island(ids++, new Coordinate(scanner.nextBigDecimal(), scanner.nextBigDecimal()), scanner.nextBigDecimal()));
        }

        /* Iterate Through Boats and Islands to Find Collisions */
        TreeMap<BigDecimal, List<Collision>> collisions = new TreeMap<>();
        for(Boat boat : boats) {
            for(Boat otherBoat : boats) {
                if(boat.id() != otherBoat.id()) {
                    BigDecimal collisionTime = boat.doesIntersect(otherBoat);
                    if(collisionTime.compareTo(BigDecimal.ZERO) >= 0) {
                        if(!collisions.containsKey(collisionTime)) {
                            collisions.put(collisionTime, new ArrayList<>());
                        }
                        collisions.get(collisionTime).add(new Collision(boat, otherBoat));
                    }
                }
            }
            for(Island island : islands) {
                BigDecimal collisionTime = boat.doesIntersect(island);
                if(collisionTime.compareTo(BigDecimal.ZERO) >= 0) {
                    if(!collisions.containsKey(collisionTime)) {
                        collisions.put(collisionTime, new ArrayList<>());
                    }
                    collisions.get(collisionTime).add(new Collision(boat, island));
                }
            }
        }

        Map<Integer, String> results = new HashMap<>();
        Set<Integer> alreadyCrashed = new HashSet<>();
        for(var entries : collisions.entrySet()) {
            for(var collision : entries.getValue()) {
                // the boat must be uncrashed
                if(collision.obj2() instanceof Island && !alreadyCrashed.contains(collision.obj1().id())) {
                    alreadyCrashed.add(collision.obj1().id());
                    results.put(collision.obj1().id(), "Island");
                }
                // both boats have to be in an uncrashed state
                if(collision.obj2() instanceof Boat && !alreadyCrashed.contains(collision.obj1().id()) && !alreadyCrashed.contains(collision.obj2().id())) {
                    alreadyCrashed.add(collision.obj1().id());
                    alreadyCrashed.add(collision.obj2().id());
                    results.put(collision.obj1().id(), "Boat");
                    results.put(collision.obj2().id(), "Boat");
                }
            }
        }

        for(var boat : boats) {
            System.out.println(results.getOrDefault(boat.id(), "Safe"));
        }
    }
}
```
