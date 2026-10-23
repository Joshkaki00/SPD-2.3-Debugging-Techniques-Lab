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

I expected a new order to show up on the homepage after submit. The homepage and the form both loaded. Submit is where it broke.

`POST /order` came back 500. Tracing backward through the traceback landed me on line 79 of `app.py`, in `pizza_order_submit`, on `PizzaTopping(topping=topping_str)`. The message was `TypeError: 'topping' is an invalid keyword argument for PizzaTopping`. I thought `topping_str` was missing, and then I thought the class only took integers. The columns are `id`, `topping_type`, and `pizza_id`. `topping` is not one of them, so I renamed the keyword to `topping_type`.

It 500'd again on the redirect. `BuildError: Could not build url for endpoint '/'`. Line 84 called `url_for('/')`. I used the route because that is what `@app.route('/')` says. `url_for` wants the function name, which is `home`. `fulfill_order` already called `url_for('home')`. I matched that.

The next try was a 302 on the POST and a 200 on the homepage, and the order was still missing. I had 302 confused with "not modified." The redirect had worked. I traced forward from `db.session.add(pizza)`. `fulfill_order` commits before it redirects. This handler never did, so the row was gone by the time the homepage queried. I added the commit.

Commit then raised `IntegrityError: NOT NULL constraint failed: pizza.order_name`. The insert parameters were `(None, None, 'THIN', 0)`, name, size, crust, fulfilled. Crust arrived. Name and size were `None`. `print(pizza.size)` had already printed `None`, so the object was bad before line 82. `request.form.get` returns `None` when the key is not the input's `name`. The code asked for `name` and `size`. The inputs were `order_name` and `pizza_size`. I renamed the inputs to match.

Once it saved, every topping was on the pizza. The loop read `for topping_str in ToppingType`, the whole enum. `toppings_list` was already there and unused, so I looped that instead.

That failed the `toppingtype` check and tried to insert `('S', 2)`. I thought a checkbox would submit a bool. It submits the value, like `SPINACH`. `get` returned that as one string, and the loop walked the letters. `getlist('toppings')` keeps each checked name whole. I switched line 70 to `getlist`. The next POST was 302, the homepage was 200, and only the toppings I checked were listed.

## Exercise 2

[[Your answer goes here!]]

## Exercise 3

[[Your answer goes here!]]
