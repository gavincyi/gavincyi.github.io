
---
layout: post
title: When do we need Pydantic?
subtitle: Pydantic is so great to use, but is it always necessary?
cover-img: https://i.pinimg.com/564x/ea/f1/5c/eaf15ca2b1c8a5fa76b95b5ff372a1a1.jpg
tags: [python, dataclass, pydantic]
comments: true
---


When do we need Pydantic?

Data class was introduced to Python 3.7 ([PEP 557](https://peps.python.org/pep-0557/)) to describe a data object with typed attributes.

```
from dataclasses import dataclass

@dataclass
class Order:
  “””
  Order class describing price and quantity.
  “””
  price: float
  quantity: int
  active: bool
```

The declaration is equivalent to cpp-like [direct initialisation](https://en.cppreference.com/w/cpp/language/direct_initialization)

```
def __init__(price: float, quantity: int, active: bool):
    self.price = price
    self.quantity = quantity
    self.active = active
```

Type declaration in attributes suggests hints to users and developers, but has no enforcement on the input arguments. Developers are only benefited with the brevity of the implicit constructor. Not to mention the data class is only complicated when private attributes are needed.

## Inheritance could be headache

If you are going to introduce inheritance in data models, it could be a headache in dataclasses.

For example, the base model `Order` contains an optional attribute `active`, and the attribute is defaulted as `True`.

```
@dataclass
class Order:
  “””
  Order class describing price and quantity.
  “””
  price: float
  quantity: int
  active: bool = True
```

and a subclass model `StopOrder` has an additional attribute `stop_price`

```
@dataclass
class StopOrder(Order):
  “””
  Stop order with a specified stop price.
  “””
  stop_price: float
```

You will fail to construct the class with an error `TypeError: non-default argument 'stop_price' follows default argument` if you use Python 3.9 or below. The short answer is Python attempted to construct a constructor `__init__` that a default argument is between the non-default arguments, like below.

```
def __init__(self, price: float, quantity: int, active: bool = True, stop_price: float):
    …
```

The above issue was then resolved in Python 3.10+, but it means dataclass is not well-defined until later versions.

## Pydantic

[Pydantic](https://docs.pydantic.dev/) is the backbone of [FastAPI](https://fastapi.tiangolo.com/) and is designed to provide a more complete framework around dataclasses. The major feature is the attribute typing hints are enforced, and users can provide custom validators per data attribute.

```
from pydantic import BaseModel

class Order(BaseModel):
  “””
  Order class describing price and quantity.
  “””
  price: float
  quantity: int
  active: bool

# Gives an integer 1 rather than ‘1’
order = Order(price=100.0, quantity=‘1’, active=True)
order.quantity
# Out: 1

# Throws Validation error on the string input of ‘quantity’
Order(price=100.0, quantity=‘Peter’, active=True)
```

The object model guarantees the object is validated before passing to downstream. A huge improvement for users and developers.

## Model conversion

Same as `dataclasses.dataclass`, Pydantic can convert models into native dict / json

```
order.dict(). # {'price': 100.0, 'quantity': 1, 'active': True}
order.json()  # '{"price": 100.0, "quantity": 1, "active": true}'
```

In the meantime, Pydantic model can be converted from [“arbitrary class instance”](https://docs.pydantic.dev/usage/models/#orm-mode-aka-arbitrary-class-instances). Of course it is not entirely “arbitrary”, but as long as the class instance contains the same set of attributes, Pydantic attempts to convert it into Pydantic model.

For example, another order class is defined with `namedtuple`.

```
from collections import namedtuple
OrderModel = namedtuple("OrderModel", ["price", "quantity", "active"])
```

and the Pydantic model has activated the ORM mode. The model creation can be achieved by calling class method `from_orm`

```
from pydantic import BaseModel

class Order(BaseModel):
  “””
  Order class describing price and quantity.
  “””
  price: float
  quantity: int
  active: bool

  class Config:
      orm_mode = True

Order.from_orm(OrderModel(price=100.0, quantity=1, active=True))
# Out: Order(price=100.0, quantity=1, active=True)
```

With ORM mode, data objects from database, e.g. imported via SQLAlchemy or Django, can be trivially converted to Pydantic model. 

## Private attributes

Pydantic provides private attributes to keep confidential data attributes hidden.

```
from pydantic import BaseModel

class Order(BaseModel):
  “””
  Order class describing price and quantity.
  “””
  price: float
  quantity: int
  active: bool
  _creator: str

  def __init__(self, creator, **kwargs):
    super().__init__(**kwargs)
    self._creator = creator

# Private attribute is hidden 
order = Order(price=100.0, quantity=‘1’, active=True, creator=“Peter”)
order.dict()
# Out: Order(price=100.0, quantity=1, active=True)
```

## Do you really need validators?

As mentioned, one of the greatest advantages of employing Pydantic is the default validator provided for each supported type.  In the meantime, actually, it is a crucial question for developers whether the default validator is needed.

For example, a data model `TBaseModel` simply takes a data attribute of an integer list `a`, while creating a new object with 10 million integers of `a` may take a few seconds to validate the full list.

```
a = list(range(10000000))

b = TBaseModel(a=a)
# %timeit 5.17 s ± 33.3 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
``` 

If your upstream already secures the data type of the data input, is enforcing validators more a bonus or just purely a painful overhead?

## Do you really need Pydantic?

I assume there are more considerations, and also a few alternative options, to choose between them.

For these considerations, a few primary and fundamental characteristics impact hugely. For example, I always prefer sticking with standard libraries in enterprise level development, as I have little interest in understanding how an application breaks down due to an arbitrary low level dependency upgrade. If possible, I would stick to dataclasses and then wrap it with custom validators. In the meantime, I have better control on the release and deployment of my open sourced applications, and I do not bother to provide any compatibility to legacy Python releases, e.g. Python 3.6. In such a case, I am happy to rely on Pydantic even if it does not serve the greatest scope of audiences, but sufficient majority.

At the moment, let me summarise the major features / differences between dataclasses and Pydantic, especially for those readers who are making a decision on choosing one of them as the model framework.

Standard library: No dependency needed for standard library dataclasses. A big win.
Supported version
Painful inheritance: Immune in Pydantic
Type validation: Rigidly enforced in Pydantic
Class decorator: dataclass is applied with decorator, but Pydantic with `BaseModel` inheritance

A short table can be summarised as below

|Feature|dataclass|Pydantic|
|:---:|:---:|:---:|
|Standard library|Y|N|
|Supported version|Python 3.7+|Python 3.7+|
|Painless inheritance|Python 3.10+|Python 3.7+|
|Type validation|N|Y|
|Dump to / load from dict|Y|Y|
|Dump to / load from arbitrary class|N|Y|
|Class decorator|Y|N|

Also, you can sort out a slightly better version of dataclass with a third party library [attr](https://www.attrs.org/en/stable/index.html), and Tin [detailed](https://threeofwands.com/why-i-use-attrs-instead-of-pydantic/) greatly why attr is sometimes more preferable than Pydantic.

## Conclusion

Let me summarise it in a "simple" decision tree.



Hope it helps you choose between them.
