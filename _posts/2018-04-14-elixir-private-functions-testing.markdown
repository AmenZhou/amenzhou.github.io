---
layout: post
title:  "How to Test Private Functions in Elixir?"
date:   2018-04-14 15:10:00 -0500
categories: elixir
tags: [test]
---

#Drafting

There are a lot of articles giving different ways to test private functions in Elixir. It is not a question only for Elixir programming language.

Basically people are saying that we shouldn't test private function **directly**. But that only means that we should not call private functions directly in the tests, we still need to guarantee the quality of private functions in the tests.

### Here is an example!

Let's say there is an `Employee` module, it has an attribute called `employee_id`. The `employee_id` is a combination of `first_name` and `last_name`.

```elixir
  defmodule Employee do
    defstruct [:first_name, :last_name, :age, :employee_id]

    def create_changeset(struct, params \\ %{}) do
      params = params |> Map.put(params |> generate_employee_id)
      changeset(struct, params)
    end

    defp generate_employee_id(%{first_name: first_name, last_name: last_name})
      "#{first_name}_#{last_name}"
    end
  end
```

We can easily test the behavior of `generate_employee_id` by this

```elixir
  test "creates an employee changeset with a correct employee_id" do
    params = %{first_name: "Haimeng", last_name: "Zhou", age: 10}
    changeset = Employee.create_changeset(%Employee{}, params)
    assert changeset.changes[:employee_id] == "Haimeng_Zhou"
  end
```

### Increase complexity of this example!

In order to make the `employee_id` unique, we add more logic

```elixir
  defmodule Employee do
    defstruct [:first_name, :last_name, :age, :employee_id]

    def create_changeset(struct, params \\ %{}) do
      params = params |> Map.put(params |> generate_employee_id)
      changeset(struct, params)
    end

    # Add a safeguard here for the recurring function
    defp generate_employee_id(%{first_name: first_name, last_name: last_name}, n) when n >= 3 do
      suffix = Enum.random(0..999) |> Integer.to_string
      employee_id = "#{first_name}_#{last_name}_#{suffix}"
    end

    defp generate_employee_id(%{first_name: first_name, last_name: last_name}, n \\ 1)
      n = Enum.random(0..999) |> Integer.to_string
      employee_id = "#{first_name}_#{last_name}_#{n}"

      !(employee_id |> is_unique_employee_id) ? generate_employee_id : employee_id
    end

    defp is_unique_employee_id(employee_id) do
      # A sql query to look up the employee_id in the employees table and then return a boolean from this function
    end
  end
```

Something I want to test from the above code

* The new employee_id is unique
* The employee_id format is corret
* The recurring function can be executed no more than 3 times

