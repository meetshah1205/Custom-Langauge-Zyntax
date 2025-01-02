# Zyntax (`.zx`)
![zyntax-logo](assets/zyntax.webp)
# What is an interpreter?
It is a program that reads and executes our code line by line.

#### Example
One interpreter is the Python Interpreter.

When we write:
```shell
python main.py
```
We invoke the interpreter on `main.py`.

Now that we have an idea of what is an interpreter, we need a langauge to interpret, of course.

# Stack based language
is the kind of language we will create.

Which means all its memory management is done by a stack,
 which is basically like an array.

It starts at `[0]` and ends at `[n-1]`.

but we can't do:
```python
stack[3] = 7
```
because stack is a last in first out data structure relying on something called a stack pointer to point to certian elements of the stack.

and do only 2 operations, push and pop.

So if we say:
```shell
PUSH 10
``` 
it will move the pointer up one index and put 10 there.

We can do similar like:
```shell
PUSH 7
```
This is put 7 1 index above `10`.
The `POP` operation is the opposite,
```shell
POP 7
```
Will point the stack pointer down 1 index, effectively forgetting that the element before that existed.

Now that we know how that works, lets look our language.

# Our language
![our-language](assets/our-language.jpg)

You are already familier with 2 instructions, `PUSH` and `POP`, but lets go a bit deeper there and also take a look at other insturctions.
- `PUSH`
> OPCODE NUMBER: `PUSH` number to top of the stack.

- `POP`
> OPCODE: `POP` number from stack and return it.

- `ADD`
> OPCODE: `POP` 2 numbers from the stack and `PUSH` the sum.

- `SUB`
> OPCODE: `POP` 2 numbers from the stack and subtracts the first from the second.

- `PRINT`
> OPCODE `STRING_LITERAL`: Prints the `string_literal` to the terminal.

- `READ`
> OPCODE: Read number from IO Input and `PUSH` it back to the stack.

- `JUMP.EQ.0`
> OPCODE LABEL: Jump to the label if top of the stack is 0

- `JUMP.GT.0`
> OPCODE LABEL: Jump to the label if top of the stack is greater than 0.

## Example program of our language
### Identity crisis of numbers
```shell
READ
READ
SUB
JUMP.EQ.0 L1
PRINT not equal
HALT

L1:
PRINT equal
HALT
```

#### Explaination
- We get 2 numbers from the user.
```shell
READ
READ
```
- Subtract them from each other
```shell
SUB
```

- Check the top of the stack if it is equal to 0, if it isn't equal to 0 we jump to the next line and print `"not euqal"` and exit
```shell
JUMP.EQ.0 L1
PRINT not equal
HALT
```

* But if it equal to 0 we jump to L1.
```shell
JUMP.EQ.0 L1
``` 

* Print `"equal"` and exit
```shell
L1:
PRINT equal
HALT
```

If we were to create this in Python it would probably be something like:
```python
x = int(input())
y = int(input())
def L1():
    print("equal")
    exit
if x - y != 0:
    print("not equal")
    exit
else:
    L1()
```

###
```shell
READ
JUMP.EQ.0 L1

LOOP:
PUSH 2
SUB
JUMP.EQ.0 L1
JUMP.GT.0 LOOP
PRINT odd
HALT

L1:
PRINT even
HALT
```

#### Explaination
```shell
READ # Read the user input and put it on stack
JUMP.EQ.0 L1 # If the input is 0, jump to L1

LOOP:
PUSH 2 # If not push 2 to the stack
SUB # Subtract the user input from 2
JUMP.EQ.0 L1 # If it is 0 jump to L1
JUMP.GT.0 LOOP # Continue the loop if it isn't 0
PRINT odd # Print odd if it is less than 0, like -1
HALT # Exit

L1: # The L1 function
PRINT even # Print even as this function is only called if the number is even
HALT # Exit
```

# Creating interpreter
I will be using python but i am planning on converting it to something like C or Java later.
## Parsing
The first thing we do is if we retreave the program file path from the command line arguement.
```python
import sys

# read arguements
program_filepath = sys.argv[1]
```

Then we tokenize the program.
```python
######################################
#           Tokenize Program        #
######################################

# read the file
with open(program_filepath, 'r') as program_file:
    program_lines = [line.strip() for line in program_file.readlines()]

program = []
token_counter = 0 # Track where we are in the program
```
We convert the lines into an array of tokens.

Then we rertrieve the opcode by looping over the program adn process it.

```python
for line in program_lines:
    parts = line.split(" ")
    opcode = parts[0]

    # check for empty lines
    if opcode == " ":
        continue

    # Check if its a label
    if opcode.endswith(":"):
        label_tracker[opcode[:-1]] = token_counter
        continue

    # check opcode token
    program.append(opcode)
    token_counter += 1

    # handle each opcode
    if opcode == "PUSH":
        # expecting a number
        number = int(parts[1])
        program.append(number)
        token_counter += 1
    elif opcode == "PRINT":
        # parse string literal
        string_literal = " ".join(parts[1:])
        program.append(string_literal)
        token_counter += 1
    elif opcode == "JUMP.EQ.0":
        # read label
        label = parts[1]
        program.append(label)
        token_counter += 1
    elif opcode == "JUMP.GT.0":
        # read label
        label = parts[1]
        program.append(label)
        token_counter += 1

print(program) # Testing
```

If we run:
```shell
python interpreter.py Examples/program1.zx
```
It gives us this output:
```shell
['READ', 'READ', 'SUB', 'JUMP.EQ.0', 'L1', 'PRINT', '"not equal"', 'HALT', '', 'PRINT', '"equal"', 'HALT']
```
Which is the list of tokens we used in our program.

Now we have to create our stack to execute this stuff.

## Stack
For that we create a class named `Stack` which has a constructer that takes in the size.

It will initialize the stack pointer and basically do a lot.
```python
class Stack:
    def __init__(self, size):
        self.buf = [0 for _ in range(size)]
        self.sp = -1 # Stack pointer
```

Then we create the push function and pop function.
```python
    def push(self, number):
        self.sp += 1
        self.buf[self.sp] = number
    
    def pop(self):
        number = self.buf[self.sp]
        self.sp -= 1
        return number
```
Then we create a `top` function which doesn't necesarrily do anything but return the top element of the stack.
```python
def top(self):
        return self.buf[self.sp]
```

Now that we have created a class for stack, lets actually initialize it.
```python
pc = 0 # program counter
stack = Stack(256)
```
Now that we have our stack, we need to keep running the program until we encounter the `HALT` keyword.
```python
while program[pc] != "HALT":
    opcode = program[pc]
    pc += 1

    if "opcode" == "PUSH":
        number = program[pc]
        pc += 1
        stack.push(number)

    elif opcode == "POP":
        stack.pop()
    
    elif opcode == "ADD":
        a = stack.pop()
        b = stack.pop()
        stack.push(a + b)
    
    elif opcode == "SUB":
        a = stack.pop()
        b = stack.pop()
        stack.push(b - a)
    
    elif opcode == "PRINT":
        string_literal = program[pc]
        pc += 1
        print(string_literal)
    
    elif opcode == "READ":
        number = int(input())
        stack.push(number)

    elif opcode == "JUMP.EQ.0":
        number = stack.pop()
        if number == 0:
            pc = label_tracker[program[pc]]
        else:
            pc += 1
    
    elif opcode == "JUMP.GT.0":
        number = stack.top()
        if number > 0:
            pc = label_tracker[program[pc]]
        else:
            pc += 1
```

Now its time for testing.

Run the command:
```shell
python interpreter.py path/to/your/file.zx
```

