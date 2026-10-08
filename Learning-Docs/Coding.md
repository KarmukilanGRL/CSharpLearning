## Sample Coding:

### Reverese String

#### using for loop

string str = "Hello";
string reversed = "";

for (int i = str.Length - 1; i >= 0; i--)
{
    reversed += str[i];
}

Console.WriteLine(reversed);

#### Using `char[]`

string str = "Hello";

char[] chars = str.ToCharArray();

int left = 0;
int right = chars.Length - 1;

while (left < right)
{
    char temp = chars[left];
    chars[left] = chars[right];
    chars[right] = temp;

    left++;
    right--;
}

string reversed = new string(chars);

Console.WriteLine(reversed);

#### Using LINQ

string str = "Hello";

string reversed = new string(str.Reverse().ToArray());

Console.WriteLine(reversed);

### Finding Duplicate character:

#### Dictionary

string str = "programming";

Dictionary<char, int> frequency = new Dictionary<char, int>();

foreach (char c in str)
{
    if (frequency.ContainsKey(c))
        frequency[c]++;
    else
        frequency[c] = 1;
}

foreach (var item in frequency)
{
    if (item.Value > 1)
        Console.WriteLine($"{item.Key} -> {item.Value}");
}

#### TryGetValue:

string str = "programming";

Dictionary<char, int> frequency = new();

foreach (char c in str)
{
    frequency.TryGetValue(c, out int count);
    frequency[c] = count + 1;
}

foreach (var item in frequency)
{
    if (item.Value > 1)
        Console.WriteLine($"{item.Key} -> {item.Value}");
}

### Swap two integer:

#### tuple deconstruction

int a = 10;
int b = 20;

(a, b) = (b, a);

Console.WriteLine(a); // 20
Console.WriteLine(b); // 10

#### Arithmetic approach

int a = 10;
int b = 20;

a = a + b;
b = a - b;
a = a - b;

#### XOR approach:

int a = 10;
int b = 20;

a = a ^ b;
b = a ^ b;
a = a ^ b;

### Palindrome:

#### Two pointers

string input = "madam";

int left = 0;
int right = input.Length - 1;

bool isPalindrome = true;

while (left < right)
{
    if (input[left] != input[right])
    {
        isPalindrome = false;
        break;
    }

    left++;
    right--;
}

Console.WriteLine(isPalindrome);

### Anagram:

#### Dictionary approach

static bool IsAnagram(string s1, string s2)
{
    if (s1.Length != s2.Length)
        return false;

    Dictionary<char, int> frequency = new();

    foreach (char c in s1)
    {
        if (frequency.TryGetValue(c, out int count))
            frequency[c] = count + 1;
        else
            frequency[c] = 1;
    }

    foreach (char c in s2)
    {
        if (!frequency.TryGetValue(c, out int count))
            return false;

        frequency[c] = count - 1;
    }

    foreach (int count in frequency.Values)
    {
        if (count != 0)
            return false;
    }

    return true;
}

### Remove Duplicate Characters:

#### HashSet

string input = "programming";

HashSet<char></char>< char > seen = new();
StringBuilder result = new();

foreach (char c in input)
{
    if (seen.Add(c))
    {
        result.Append(c);
    }
}

Console.WriteLine(result.ToString());

#### LINQ

string result = new string(input.Distinct().ToArray());

### Largest & Smallest Number in an Array:

int[] numbers = { 10, 5, 25, 8, 30, 2 };

int largest = numbers[0];
int smallest = numbers[0];

for (int i = 1; i < numbers.Length; i++)
{
    if (numbers[i] > largest)
        largest = numbers[i];

    if (numbers[i] < smallest)
        smallest = numbers[i];
}

Console.WriteLine($"Largest: {largest}");
Console.WriteLine($"Smallest: {smallest}");

### Second Largest Element

int[] numbers = { 10, 5, 25, 8, 30, 2 };

int largest = int.MinValue;
int secondLargest = int.MinValue;

foreach (int number in numbers)
{
    if (number > largest)
    {
        secondLargest = largest;
        largest = number;
    }
    else if (number > secondLargest && number != largest)
    {
        secondLargest = number;
    }
}

Console.WriteLine($"Largest: {largest}");
Console.WriteLine($"Second Largest: {secondLargest}");

### Count Words in a String:

string input = "Hello world from C sharp";

int count = 0;
bool insideWord = false;

foreach (char c in input)
{
    if (!char.IsWhiteSpace(c))
    {
        if (!insideWord)
        {
            count++;
            insideWord = true;
        }
    }
    else
    {
        insideWord = false;
    }
}

Console.WriteLine(count);

#### using Split()

int count = input
    .Split(' ', StringSplitOptions.RemoveEmptyEntries)
    .Length;

### FizzBuzz:

#### Question

Print numbers from `1` to `N`.

Rules:

* Divisible by **3** → print `Fizz`
* Divisible by **5** → print `Buzz`
* Divisible by **both 3 and 5** → print `FizzBuzz`
* Otherwise → print the number
* `N = 15`

#### Solution:

int n = 15;

for (int i = 1; i <= n; i++)
{
    if (i % 3 == 0 && i % 5 == 0)
    {
        Console.WriteLine("FizzBuzz");
    }
    else if (i % 3 == 0)
    {
        Console.WriteLine("Fizz");
    }
    else if (i % 5 == 0)
    {
        Console.WriteLine("Buzz");
    }
    else
    {
        Console.WriteLine(i);
    }
}

### Reverse an Array:

#### Two-pointer approach - in place

int[] numbers = { 1, 2, 3, 4, 5 };

int left = 0;
int right = numbers.Length - 1;

while (left < right)
{
    (numbers[left], numbers[right]) =
        (numbers[right], numbers[left]);

    left++;
    right--;
}

Console.WriteLine(string.Join(" ", numbers));

#### Built-in

Array.Reverse(numbers);

### Find the Missing Number:

#### Sum formula

int[] numbers = { 3, 0, 1 };

int n = numbers.Length;

int expectedSum = n * (n + 1) / 2;

int actualSum = 0;

foreach (int number in numbers)
{
    actualSum += number;
}

int missing = expectedSum - actualSum;

Console.WriteLine(missing);

#### XOR

how it works, XOR all n integeres and XOR with XOR of actual elements
0^1^2^3^3^0^1

same value cancels each other - only 2 remains

int[] numbers = { 3, 0, 1 };

int result = numbers.Length;

for (int i = 0; i < numbers.Length; i++)
{
    result ^= i;
    result ^= numbers[i];
}

Console.WriteLine(result);
