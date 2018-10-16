# Euclidean Algorithm

The Euclidean algorithm is used to get the greatest common divisor (GCD) of two
numbers.

## Basic Algorithm

For two given numbers x, y, take the larger number and subtract the other
number. Repeat this process until both numbers are equal, this number is the
GCD of both numbers.

#### Example, get the GCD of 112 and 196

```
112  196      (196 - 112)  
112   84       (112 - 84)  
 28   84        (84 - 28)  
 28   56        (56 - 28)  
 28   28  
```

## Improved Algorithm (Thanks Gabriel Lamé)

For two given numbers x, y, take the larger number and divide it by the smaller
number and keep its remainder. Repeat this process until one of the numbers is
zero. The GCD will be the non-zero number.

#### Example, get the GCD of 112 and 196

```
112  196      (196 % 112)  
112   84       (112 % 84)  
 28   84        (84 % 28)  
 28    0  
```
