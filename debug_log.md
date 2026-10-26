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

I used trace backward on the crashes. I started at the last frame in my file and walked to the bad name or the bad value. I used trace forward after the redirect worked and the homepage stayed empty. I followed `pizza_order_submit` from `db.session.add(pizza)` and from `print(pizza.size)`.

I expected a new order to show up on the homepage after submit. The homepage and the form both loaded. Submit is where it broke.

`POST /order` came back 500. Tracing backward through the traceback landed me on line 79 of `app.py`, in `pizza_order_submit`, on `PizzaTopping(topping=topping_str)`. The message was `TypeError: 'topping' is an invalid keyword argument for PizzaTopping`. I thought `topping_str` was missing, and then I thought the class only took integers. The columns are `id`, `topping_type`, and `pizza_id`. `topping` is not one of them, so I renamed the keyword to `topping_type`.

It 500'd again on the redirect. `BuildError: Could not build url for endpoint '/'`. Line 84 called `url_for('/')`. I used the route because that is what `@app.route('/')` says. `url_for` wants the function name, which is `home`. `fulfill_order` already called `url_for('home')`. I matched that.

The next try was a 302 on the POST and a 200 on the homepage, and the order was still missing. I had 302 confused with "not modified." The redirect had worked. I traced forward from `db.session.add(pizza)`. `fulfill_order` commits before it redirects. This handler never did, so the row was gone by the time the homepage queried. I added the commit.

Commit then raised `IntegrityError: NOT NULL constraint failed: pizza.order_name`. The insert parameters were `(None, None, 'THIN', 0)`, name, size, crust, fulfilled. Crust arrived. Name and size were `None`. `print(pizza.size)` had already printed `None`, so the object was bad before line 82. `request.form.get` returns `None` when the key is not the input's `name`. The code asked for `name` and `size`. The inputs were `order_name` and `pizza_size`. I renamed the inputs to match.

Once it saved, every topping was on the pizza. The loop read `for topping_str in ToppingType`, the whole enum. `toppings_list` was already there and unused, so I looped that instead.

That failed the `toppingtype` check and tried to insert `('S', 2)`. I thought a checkbox would submit a bool. It submits the value, like `SPINACH`. `get` returned that as one string, and the loop walked the letters. `getlist('toppings')` keeps each checked name whole. I switched line 70 to `getlist`. The next POST was 302, the homepage was 200, and only the toppings I checked were listed.

## Exercise 2

I used trace backward on this one. I started at each traceback and walked to the value that line was reading.

I expected a city and a unit choice to come back as a weather page. The app would not even start. `import requests` on line 2 pulled in `urllib3`, and the last frame was `ModuleNotFoundError: No module named 'urllib3.packages.six.moves'`. The packages were installed. `requirements.txt` had `urllib3==1.26.2`, and the venv is Python 3.13. I upgraded `requests` to 2.34.2 and `urllib3` to 2.8.0.

Submitting San Francisco in Fahrenheit then 500'd on line 52, `KeyError: 'name'`. I printed `result_json` and got `{'cod': '400', 'message': 'Nothing to geocode'}`. The browser URL was `/results?city=San+Francisco&units=imperial`. The handler was calling `get('users_city')` and `get('requested_units')`, so both values were `None`. I first renamed the API params to those same wrong names, which did not help. The `get` strings have to match the URL. I also tried `get('place')`. `place` is not on the URL either. `get('city')` and `get('units')` are the ones that match.

The city was finally in the variable, and the API still answered `Nothing to geocode`. `params` was sending it as `place`. The current weather docs call the city parameter `q`, as in `q={city name}`. I changed that key to `q`. The next print had `'name': 'San Francisco'` and `'cod': 200`.

The crash moved to line 55, `KeyError: 'temperature'`. Inside `main` the print showed `temp`, `feels_like`, `temp_min`, and the rest. No `temperature`. I read `temp` instead. The page loaded at about 65 F.

Celsius came back around 18 with wind speed 7.72. Kelvin came back around 291 with the same wind speed 7.72. Fahrenheit wind was 17.27. The results template always wrote `miles/hour` after the wind number, which only fits the Fahrenheit speed. Metric and Kelvin wind from this API is meters per second. I added `get_wind_units`, same shape as `get_letter_for_units`, passed `wind_units` in the context, and printed `{{ wind_units }}` in the template. All three submits then returned 200.

## Exercise 3

I used trace backward on the crashes. I started at the last frame in `utils.py` and checked the index or the type that line used. I used divide and conquer on the merge. I printed the two halves after the recursive calls. If the halves were already sorted and `arr` was not, I only looked at the merge loops.

I expected `merge_sort` to turn `[7, 4, 9, 3, 5, 6, 1, 2]` into `[1, 2, 3, 4, 5, 6, 7, 9]`, then `binary_search` to return index `4` for `5`. The first run died with `IndexError` in `merge_sort`. The traceback showed line 16 and line 37. Line 16 is just `merge_sort(left_side)` calling itself. Line 37 was the crash, `arr[k] = right_side[i]`.

I printed `i`, `j`, `k`, and `right_side` above that write. One hit was `i 1 j 0 k 1 right [4]`. The `while` only promises `j` is in range. `right_side[1]` does not exist. I changed it to `right_side[j]`. I also added `k += 1` after leftover writes because `k` is the write slot in `arr`.

`merge_sort` then finished and printed `[9, 7, 6, 5, 4, 3, 2, 1]`. `binary_search` crashed on line 53, `TypeError: list indices must be integers or slices, not float`. Line 50 was `mid = (high + low) / 2`. In Python 3, `/` makes a float. I switched it to `//`, same as the midpoint in `merge_sort`. I printed `mid` and `type(mid)` so I could see `<class 'int'>`.

The list was still high to low. Line 23 used `> `, so the bigger front value was written first. I changed it to `<=` so the smaller one wins. The next run looked worse, `[1, 2, 3, 4, 6, 2, 9, 2]`, and the index of `5` was `-1`. The halves print showed `[5, 6]` and `[1, 2]`, already sorted, then `i 2 j 2 k 2`. Both sides were consumed and `k` had only moved twice. The leftover left loop still did not bump `k`, so `6` overwrote `5`. I added `k += 1` there too.

After that the list was `[1, 2, 3, 4, 5, 6, 7, 9]` and `k` was `8` on the last merge. `binary_search` still returned `-1`. The mid prints were `3` then `5`. `arr[3]` is `4`, so `low` became `4`. `arr[5]` is `6`, so `high` became `4`. `5` sits at index `4`. The loop was `while low < high`, and `4 < 4` is false, so it never read that last slot. I changed it to `while low <= high`. The next run printed `mid 4` and `Index of 5 is: 4`.
