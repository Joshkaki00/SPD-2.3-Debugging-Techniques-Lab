# Debug Log

**Explain how you used the the techniques covered (Trace Forward, Trace Backward, Divide & Conquer) to uncover the bugs in each exercise. Be specific!**

In your explanations, you may want to answer:

- What is the expected vs. actual output?
- If there is a stack trace, what useful information does it contain?
- Which technique did you use, on which line numbers?
- What assumptions did you have about each line of code, and which ones were proven to be wrong?

_Example: I noticed that the program should show pizza orders once a new order is made, and that it wasn't showing any. So, I used the trace forward technique starting on line 13. I discovered the bug on line 27 was caused by a typo of 'pzza' instead of 'pizza'._

_Then I noticed another bug ..._

## Exercise 1

I expected a submitted order to be saved and then show up on the homepage. The homepage loaded, and the order form loaded, but submitting the form did not save the order. I worked through one crash at a time. Most of these were trace backward: the traceback named my file and the line, and the last line of the traceback said what was actually wrong. One of them was trace forward from a value I printed.

First crash: `POST /order` returned 500. The frame in my file was `app.py` line 79, `pizza.toppings.append(PizzaTopping(topping=topping_str))`. The error was `TypeError: 'topping' is an invalid keyword argument for PizzaTopping`. I assumed the problem was the value `topping_str`, or that the class only accepted integers. Both were wrong. The keyword on the left of the `=` was `topping`, and `PizzaTopping` has `id`, `topping_type`, and `pizza_id`. I changed the keyword to `topping_type`.

Next crash, still a 500, now at the redirect. The error was `BuildError: Could not build url for endpoint '/'`. I assumed `'/'` was correct because that is the homepage route. `url_for` takes the function name, not the route. The homepage function is `home`, and `fulfill_order` already calls `url_for('home')`. I changed line 84 to `url_for('home')`.

After that, the submit returned 302 and the homepage returned 200, but the new order was not on the page. I thought 302 meant "not modified." It means the redirect worked. I traced the handler forward from `db.session.add(pizza)`. `fulfill_order` calls `db.session.commit()` before it redirects, and `pizza_order_submit` never did. The order was only staged, so the next request could not see it. I added the commit.

The commit then raised `IntegrityError: NOT NULL constraint failed: pizza.order_name`. The useful part was the parameters: `(None, None, 'THIN', 0)`, which is order name, size, crust, and fulfilled. Crust came through. Name and size were `None`. I also had a `print(pizza.size)` that printed `None` before this, so the bad values were already on the object before the commit. `request.form.get` returns `None` when the key does not match an input's `name`. The handler asked for `'name'` and `'size'`. The form had `order_name` and `pizza_size`. I changed the form names to `name` and `size` so they match the `get` calls.

The order saved, but every topping was on it. The loop was `for topping_str in ToppingType`, which is every topping in the enum. `toppings_list` was already being read from the form and never used. I pointed the loop at `toppings_list`.

That crashed with `CHECK constraint failed: toppingtype` and parameters `('S', 2)`. I assumed a checkbox would come through as a bool. It comes through as the `value` string, like `SPINACH`. `request.form.get` returns one string, and a `for` loop over a string yields one letter at a time, so the first insert was `'S'`. `request.form.getlist('toppings')` returns the whole list of checked names. I switched line 70 to `getlist` and left the loop. The next submit was a 302, then a 200, and the homepage showed only the toppings I checked.

## Exercise 2

[[Your answer goes here!]]

## Exercise 3

[[Your answer goes here!]]
