'''csharp
using System;
using System.Collections.Generic;
using System.Linq;

public static class Forth
{
public static string Evaluate(string[] instructions)
{
var stack = new Stack();
var customWords = new Dictionary<string, List>(StringComparer.OrdinalIgnoreCase);

    foreach (var line in instructions)
    {
        var tokens = line.Split(' ', StringSplitOptions.RemoveEmptyEntries);
        if (tokens.Length == 0) continue;

        if (tokens[0] == ":")
        {
            if (tokens.Length < 4 || tokens[^1] != ";")
                throw new InvalidOperationException("Invalid word definition.");

            var wordName = tokens[1];
            if (int.TryParse(wordName, out _))
                throw new InvalidOperationException("Cannot redefine numbers.");

            var definition = new List<string>();
            for (int i = 2; i < tokens.Length - 1; i++)
            {
                var token = tokens[i];
                if (customWords.TryGetValue(token, out var existingDef))
                {
                    definition.AddRange(existingDef);
                }
                else
                {
                    definition.Add(token.ToUpperInvariant());
                }
            }

            customWords[wordName] = definition;
        }
        else
        {
            ExecuteTokens(tokens, stack, customWords);
        }
    }

    return string.Join(" ", stack.Reverse());
}

private static void ExecuteTokens(IEnumerable<string> tokens, Stack<int> stack, Dictionary<string, List<string>> customWords)
{
    foreach (var token in tokens)
    {
        if (customWords.TryGetValue(token, out var definition))
        {
            ExecuteTokens(definition, stack, customWords);
        }
        else if (int.TryParse(token, out int number))
        {
            stack.Push(number);
        }
        else
        {
            switch (token.ToUpperInvariant())
            {
                case "+":
                    EnsureStackCount(stack, 2);
                    stack.Push(stack.Pop() + stack.Pop());
                    break;

                case "-":
                    EnsureStackCount(stack, 2);
                    int subtrahend = stack.Pop();
                    int minuend = stack.Pop();
                    stack.Push(minuend - subtrahend);
                    break;

                case "*":
                    EnsureStackCount(stack, 2);
                    stack.Push(stack.Pop() * stack.Pop());
                    break;

                case "/":
                    EnsureStackCount(stack, 2);
                    int divisor = stack.Pop();
                    if (divisor == 0)
                        throw new DivideByZeroException();
                    int dividend = stack.Pop();
                    stack.Push(dividend / divisor);
                    break;

                case "DUP":
                    EnsureStackCount(stack, 1);
                    stack.Push(stack.Peek());
                    break;

                case "DROP":
                    EnsureStackCount(stack, 1);
                    stack.Pop();
                    break;

                case "SWAP":
                    EnsureStackCount(stack, 2);
                    int top = stack.Pop();
                    int second = stack.Pop();
                    stack.Push(top);
                    stack.Push(second);
                    break;

                case "OVER":
                    EnsureStackCount(stack, 2);
                    int first = stack.Pop();
                    int below = stack.Peek();
                    stack.Push(first);
                    stack.Push(below);
                    break;

                default:
                    throw new InvalidOperationException($"Unknown word: {token}");
            }
        }
    }
}

private static void EnsureStackCount(Stack<int> stack, int required)
{
    if (stack.Count < required)
        throw new InvalidOperationException("Insufficient values on the stack.");
}
'''
