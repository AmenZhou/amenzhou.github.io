---
layout: post
title:  "How to Test Private Functions in Elixir?"
date:   2018-04-14 15:10:00 -0500
categories: elixir
tags: [test]
---

In this post, I am not going to talk how many approaches we can use to test private functions, I am going to give an example and my solution.

Basically saying that we shouldn't test private function **directly**. That increase the complexity for writting tests, but that is not the bad thing. Most of the time, when we find ourselves are struggling on how to test a private function, it means that the opportunity to optimize our code is coming.

## Here is an example!

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

## Increase complexity of this example!

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
      "#{first_name}_#{last_name}_#{suffix}"
    end

    defp generate_employee_id(%{first_name: first_name, last_name: last_name} = params, n \\ 1)
      suffix = Enum.random(0..999) |> Integer.to_string
      employee_id = "#{first_name}_#{last_name}_#{suffix}"

      !(employee_id |> is_unique_employee_id) ?
        params |> generate_employee_id(n + 1) : employee_id
    end

    defp is_unique_employee_id(employee_id) do
      # A sql query to look up the employee_id in the employees table and then return a boolean from this function
      query = from e in Employee,
        where: e.employee_id == ^employee_id
      !(query |> Repo.all |> List.any?)
    end
  end
```

Something I want to test upon to the above code, I want to make sure that

* The new employee_id is unique (it is in a nested private function)
* The recurring function can be executed no more than 3 times
* The employee_id format is corret

**It is impossible to test the first 2 points so I am going to do some surgeries.**
### Firstly, move those 3 employee_id generating functions to a separate file

Those 3 `employee_id` correlated functions can be decoupled from `Employee` module. I create a new module.

```elixir
  defmodule EmployeeIdGenerator do
    # Add a safeguard here for the recurring function
    def generate_employee_id(%{first_name: first_name, last_name: last_name}, n) when n >= 3 do
      ...
    end

    def generate_employee_id(%{first_name: first_name, last_name: last_name}, n \\ 1)
      ...
    end

    defp is_unique_employee_id(employee_id) do
      ...
    end
  end

  defmodule Employee do
    defstruct [:first_name, :last_name, :age, :employee_id]

    def create_changeset(struct, params \\ %{}) do
      params = params |> Map.put(params |> EmployeeIdGenerator.generate_employee_id)
      changeset(struct, params)
    end
  end
```
**The `generate_employee_id/2` is a public function**

Now I can concentrate on testing the functions which I care about!

That is not enough, I am not able to test the recurring function.

### Secondly, utilize Mix.env to hack the `Enum.random`

```elixir
  defmodule EmployeeIdGenerator do
    # Add a safeguard here for the recurring function
    def generate_employee_id(%{first_name: first_name, last_name: last_name}, n) when n >= 3 do
      "#{first_name}_#{last_name}_#{suffix}"
    end

    def generate_employee_id(%{first_name: first_name, last_name: last_name} = params, n \\ 1)
      employee_id = "#{first_name}_#{last_name}_#{suffix(n)}"

      !(employee_id |> is_unique_employee_id) ?
        params |> generate_employee_id(n + 1) : employee_id
    end

    defp suffix(n \\ 1)
      case Mix.env do
        :test -> n |> Integer.to_string
        _ ->
          Enum.random(0..999)
            |> Integer.to_string
      end
    end

    defp is_unique_employee_id(employee_id) do
      ...
    end
  end
```

Yey! The `suffix` can be overwritten by test code.

### Lastly, we put them together.

In the `Employee` test file

```elixir
```




