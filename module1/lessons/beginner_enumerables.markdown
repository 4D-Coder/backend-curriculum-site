---
title: Beginner Enumerables
length: 120
tags: enumerables, map, select, find, each
---

## Learning Goals

* Learn how to use & recreate the functionality of `.map`, `.select` and `.find` using `.each`
* Understand when to use `.map`, `.select` and `.find` appropriately.

## Slides

Available [here](../slides/beginner_enumerables)

## Vocabulary  
* enumerable  
* iterate  
* map, find, find_all
* return value

## Warm Up  
* What is an **enumerable**? What is a use case for one?  
* In your notebook, write the block of code to use `.each` to print each letter in the following array `dynasty_1 = ["K", "e", "n", "n", "e", "d", "y"]`

## Intro

We've already learned how to use each, and we can do some really cool
things with it, but let's do better.

### `map` / `collect`

`map` is a lot like `each`.

The difference is that `map` actually _returns whatever we do in the block_. Think about how this is different from each which will always return the content of the _original_ array.

Let's look at this in code. Taking an array of the numbers, we want to end up with an array with the doubles of each of those numbers. With `each`, we would do it like this:

```ruby
def double                    # define a method called double
  numbers = [1, 2, 3, 4, 5]   # declare a numbers variable with the value of an array

  result = []                 # declare a variable, results, with the value of an empty array

  numbers.each do |num|       # use `.each` to iterate over the numbers array
    result << num * 2         # shovel the current number x 2 into the result array
  end                         # end the `.each` method

  result                      # return the result array
end                           # end the double method

result
=> [2, 4, 6, 8, 10]           # our array of doubles

numbers
=> [1, 2, 3, 4, 5]            # numbers array, which is unchanged
```

We've written a method called `double`. We start off with `result`, which we set to an empty array. With each, we iterate through each item of `numbers`, and with each element, we are doubling the number and we are putting it into the `result` array. At the end, we are returning the `result` variable, which should now contain [2, 4, 6, 8, 10].

This code is decent. But there are things about it I'm not entirely thrilled about. For example, we are temporarily storing things in a variable, `result`, which is inefficient. You want to avoid the use of unnecessary variables when you can. This is how we can achieve the same result using `.map`.

```ruby
def double                # define a method called double
  numbers = [1, 2, 3, 4, 5]   # declare a numbers variable with the value of an array

  numbers.map do |num|    # iterate over numbers with `.map`
    num * 2               # number x 2
  end                     # end the `.map` method

end                       # end the double method

double
=> [2, 4, 6, 8, 10]       # our numbers array has been changed!
```

Instinctually, this should look better to you. We don't have any unnecessary variable assignment and `map` is handling all we need to return.

```ruby
def double
  numbers = [1, 2, 3 ,4, 5]

  numbers.map do |num|
    num * 2
    puts "I really love math"
  end

end
```

#### Discuss:
* With this code, what do you think this method returns? Why?  
* Many folks are tempted to save the result of map to a variable. Why might I disagree with this choice?  

#### Independent Practice
The method below returns an array of the brothers names in all caps; your job is to write one using the `map` method. (Touch an `enums_practice.rb` file in your M1 directory and write the code in that file)

```ruby
def kennedy_brothers
  brothers = ["Robert", "Ted", "Joseph", "John"]

  caps_brothers = []

  brothers.each do |brother|
    caps_brothers << brother.upcase
  end

  caps_brothers

end
```
**Annotate**
Annotate each line of your new code the way the examples above were, to describe exactly what is happening at each line.

**Turn & Talk**  
Share you code with your neighbor.  Talk them through your annotations - be specific. What is similar/different? Why did you make the choices you made?  


### `find` / `detect`

A good thing about the methods in Ruby is that you can pretty much figure out what they do just by their name.

**Think About It**
What do you think `find` or `detect` does? What will it return?  

`find` will return the **first** item from the collection that evaluates as _truthy_.

But before we go into how that works, let's implement it with `each`.

```ruby
def find_me_first_even
  numbers = [1, 2, 3, 4, 5]

  numbers.each do |num|
    return num if num.even?
  end

end
```

We walk through the array, and upon the first number it comes across that makes the block return true, it just returns the item and we are done.

This looks like fairly simple code, but we can make it cleaner with `.find`.

```ruby
def find_me_first_even
  numbers = [1,2,3,4,5]

  numbers.find do |num|
    num.even?
  end

end
```

Oh just look at that, so nice. Remember, it will return the **first** item for which the block returns a truthy value.

#### Independent Practice  
Find the first sister over four letters in length.

```ruby
def find_sisters
  sisters = ["Rose", "Kathleen", "Eunice", "Patricia", "Jean"]

  sisters.find do |sister|
    # YOUR CODE HERE
  end
end
```
**Turn & Talk**  
Share your code with your neighbor.  Talk them through what is happening line by line - be specific. What is similar/different? Why did you make the choices you made?  

### `find_all` / `select`

We've figured out how to get _one_ matching thing out of an array, but what if we want ALL of the matching things?

Let's start by thinking about how we would do this using our old friend, `.each`.

```ruby
def all_the_odds
  numbers = [1,2,3,4,5]

  result = []

  numbers.each do |num|
    result << num if num.odd?
  end

  result

end
```

How does this look?

Not bad, but we're stuck with that `result` container that we don't like. Pay attention to this - it's a pattern we want to keep an eye out for in the future. **If we are catching things with a collector like this, there's probably a better enumerable that we can use.** So let's use `.select`.

```ruby
def all_the_odds
  numbers = [1,2,3,4,5]

  numbers.select do |num|
    num.odd?
  end

end
```

**Discuss:** Why is this better?

#### Independent Practice
Let's grab all of the Kennedy Spouses that start with a "V" using `find_all` rather than `each`:

```ruby
def named_v
  spouses = ["Jacqueline", "William", "Robert", "Peter", "Ethel", "Stephen", "Virginia", "Victoria"]

  spouses_starting_v = []

  spouses.each do |spouse|
    if spouse[0] == "V"
      spouses_starting_v << spouse
    end
  end

end
```

## Exercises
Follow the directions on [this README](https://github.com/ameseee/enums_practice) to get some more practice!


## Final CFU
* What do map, find, and select do? What do they return?
* What makes them preferable to each?   


### Homework
* Continue working on the [Kennedys Practice](https://github.com/ameseee/enums_practice)
* Work on the `map`, `find`, and `select` exercises for [Enums-Exercises](https://github.com/turingschool/enums-exercises).
* Work through [Beginner Enumerables](https://github.com/turingschool-examples/beginner_enums/) according to instructions within its README.
