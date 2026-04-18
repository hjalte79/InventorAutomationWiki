# Unexpected results .5

Last week we ran into an unexpected issue in one of our assemblies. The assembly consists of two plates bolted together using a linear bolt pattern. During assembly, we discovered that the hole patterns did not line up. By the time we reached the last hole, the centers were off by 10 mm.

At first glance this looked like a typical modeling or constraint error. However, the root cause turned out to be much more subtle.

## The setup

Both plates use a calculated pitch for their bolt patterns. The input values were identical, but the pitch was calculated in two different ways.

In the first plate, the pitch was defined directly in a parameter using this expression:

> round(length / (NumberOfHoles * 1 mm)) * 1 mm

In the second plate, the pitch was calculated using an iLogic rule:

> Parameter("pitch") = Math.Round(length / NumberOfHoles)

The results were not the same.
 - The first method produced a pitch of 243 mm.
 - The second method produced a pitch of 242 mm.

That 1 mm difference does not look significant until you realize it is repeated over ten holes. By the end of the pattern, the accumulated error was exactly 10 mm.

## Why this happens
The problem is not Inventor being inconsistent. The problem is how rounding and units are handled in different contexts.
At first glance, the parameter expression looks more complicated than necessary. Each part of that expression is there for a reason.

### Step 1: Forcing a unitless value







--------------------------------------------------

the pitch of the pattern in the first plate was calculated using a parameter with this expression:
> round(length / ( NumberOfHoles * 1 mm )) * 1 mm

In the second plate the pitch was calculated using an ilogic rule:

> Parameter("pitch") = Math.Round(length / NumberOfHoles)

the issue here is that the answers are 243 mm and 242mm. that difference of 1 mm is multiplied by 10 holes and gives a error of 10 mm at the end of the pattern.

Lets break down what is happening here. 

first of all the parameter expresion looks more complicated the it should be. This is because Inventor does a lot of units checking. And the 'round(...)' function needs a unitless argument.

At first glance this looks strange, but every part of the expression exists for a reason.
**a.** length / (NumberOfHoles * 1 mm)
Assume:

length has units of mm
NumberOfHoles is unitless
1 mm is explicitly added to force unit cancellation

What happens:
mm / (unitless × mm) → unitless

So this division produces a pure number, not a length.
If the real pitch is 242.5 mm, the expression evaluates to:
242.5   (unitless)

This step exists because Inventor’s round() function only accepts unitless values. If you try to round a length directly, Inventor will reject it.

**b.** * 1 mm
Finally, units are restored:
243 × 1 mm = 243 mm






```vb.net
Logger.Info("===== .Net default =====")

Logger.Info("======= ToEven 0 =======")

Logger.Info(Round(0.6))                                                           ' 1                                                                                                                                                     

Logger.Info(Round(0.5000000000000001))                     ' 1

Logger.Info(Round(0.50000000000000001))                   ' 0

Logger.Info(Round(0.5))                                                                        ' 0

Logger.Info(Round(0.49999999999999999))                   ' 0

Logger.Info(Round(0.4999999999999999))                    ' 0

Logger.Info(Round(0.4))                                                                        ' 0

 

Logger.Info("======= ToEven 2 =======")

Logger.Info(Round(1.6))                                                           ' 2                                                                                                                                                     

Logger.Info(Round(1.5000000000000001))                     ' 2

Logger.Info(Round(1.50000000000000001))                   ' 2

Logger.Info(Round(1.5))                                                                        ' 2

Logger.Info(Round(1.4999999999999999))                     ' 2

Logger.Info(Round(1.499999999999999))                      ' 1

Logger.Info(Round(1.4))                                                                        ' 1

 

Logger.Info("======= ToEven long run =======")

Logger.Info(Round(0.5, MidpointRounding.ToEven))                                                       ' 0

Logger.Info(Round(1.5, MidpointRounding.ToEven))                                                       ' 2

Logger.Info(Round(2.5, MidpointRounding.ToEven))                                                       ' 2

Logger.Info(Round(3.5, MidpointRounding.ToEven))                                                       ' 4

Logger.Info(Round(4.5, MidpointRounding.ToEven))                                                       ' 4

 

 

 

Logger.Info("======== C++ default =======")

Logger.Info("======= AwayFromZero =======")

Logger.Info(Round(0.6, MidpointRounding.AwayFromZero))                                                       ' 1

Logger.Info(Round(0.5000000000000001, MidpointRounding.AwayFromZero))    ' 1

Logger.Info(Round(0.50000000000000001, MidpointRounding.AwayFromZero))  ' 1

Logger.Info(Round(0.5, MidpointRounding.AwayFromZero))                                                       ' 1

Logger.Info(Round(0.49999999999999999, MidpointRounding.AwayFromZero))  ' 1

Logger.Info(Round(0.4999999999999999, MidpointRounding.AwayFromZero))    ' 0

Logger.Info(Round(0.4, MidpointRounding.AwayFromZero))                                                       ' 0

 

 

Logger.Info("======== Inventor parameter rounding =======")

 

Logger.Info(Round(4.4999995, MidpointRounding.AwayFromZero))                                                        ' 4

Logger.Info(Round(4.4999999999999999999999999, MidpointRounding.AwayFromZero))           ' 4

Logger.Info(Round(4.5, MidpointRounding.AwayFromZero))                                                                               ' 5

 

Logger.Info(Round(Round(4.4999995, 6),  MidpointRounding.AwayFromZero)) ' Rounding like Inventor parameters

 

' C++ sandbag

' https://www.w3schools.com/cpp/trycpp.asp?filename=demo_ref_math_round

 

' round errors in parameters

' https://www.autodesk.com/support/technical/article/caas/sfdcarticles/sfdcarticles/Wrong-result-of-the-Round-function-in-Inventor-Parameter-window.html

 

' https://help.autodesk.com/view/INVNTOR/2022/ENU/?guid=207cfafc-0993-9686-c1f4-1656fd94f83a
```