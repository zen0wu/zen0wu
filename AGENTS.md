# About me

My name is Zeno.
I like programming languages, various languages constructs and concepts.
I like exploring and learning new things.
I like deep thinking and convert those thinking into simple artifacts.

# My coding principles

- Abstraction is great, but not too much.
- Optimize for simplicity, but not easy.
- Default matters, opting in vs always true.

Things I like: 
- Simple and minimalist things.
- Clean code. 
- Think about various trade-offs deeply.
- Choose the simplest approach that works.

Things I don't like:
- Complex things.
- Over-engineered code/architecture.
- Unintuitive concepts.
- Tricks when designing high level architecture: It's ok to be clever locally but not globally, and cleverness should be explained.
- Nesting: Nested if-else, inner functions, etc.

# My coding styles

- Use literal and intuitive variable names.
- Variable declarations should be as close as possible to their (first) usage.
- Whenever there's a cleverness in the code block, explain it at the beginning of that block.
- Each function should look like a few code blocks concatenated and the logic between them should be streamlined.
- Only create abstraction when generalizing over at least 3 examples.

# Coding agent rules

If you're a coding agent, follow these rules strictly!

- Don't be weird: Cleverness is only allowed locally, sparsely and with clear justification.
- No fallback: You have the tendancy to add fallback/make things safe. Just let it fail and only add try/catch when absolutely needed.
- No nesting: Make serious efforts to make the logic streamlined. Only add nesting when absolutely necessary.
- Simple and dumb: Be as simple and dumb as possible. Use your judgement to try to understand each piece of the code you wrote and ask "would a human without any context understand this easily"?
- Minimum viable code: Try to examine and reflect on each part of the code (not only the code you wrote) and ask "Should this part exist"? If not, ruthless remove them.
