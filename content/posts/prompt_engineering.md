---
title: "Prompt Engineering"
summary: Learn About All Features in PaperMod
date: 2024-06-09
weight: 1
tags: ["Web3"]
author: ["Evanara Kumestra"]
---

这篇文章是吴恩达prompt engineering的笔记，https://learn.deeplearning.ai/courses/chatgpt-prompt-eng，课程使用的模型是gpt-3.5-turbo，我在gpt-4o对这些prompt重新进行了测试，发现有的例子已经不再适用，比如引起模型幻觉的prompt在新的模型上不会出现幻觉。但这个课程的prompt构建思路还是可以参考的。

# Prompting Principles

## Principle 1: Write clear and specific instructions

### Use delimiters to clearly indicate distinct parts of the input. Delimiters can be anything like: ```, """, < >, &lt;tag> &lt;/tag>.

使用分隔符来区分prompt中的不同的部分。

```python {linenos=true}
text = f"""
You should express what you want a model to do by \
providing instructions that are as clear and \
specific as you can possibly make them. \
This will guide the model towards the desired output, \
and reduce the chances of receiving irrelevant \
or incorrect responses. Don't confuse writing a \
clear prompt with writing a short prompt. \
In many cases, longer prompts provide more clarity \
and context for the model, which can lead to \
more detailed and relevant outputs.
"""

prompt = f"""
Summarize the text delimited by triple backticks \
into a single sentence.
```{text}```
"""
```

```text {linenos=true}
Providing clear, specific, and detailed instructions in prompts helps guide a model to produce the desired output and reduces irrelevant or incorrect responses.
```

### Ask for a structured output.

让LLM生成结构化数据。

```python {linenos=true}
prompt = f"""
Generate a list of three made-up book titles along \
with their authors and genres. 
Provide them in JSON format with the following keys: 
book_id, title, author, genre.
"""
```

````text {linenos=true}
Sure, here is the list of three made-up book titles in JSON format:

```json
[
    {
        "book_id": "1",
        "title": "Whispers of the Enchanted Forest",
        "author": "Elara Moonshadow",
        "genre": "Fantasy"
    },
    {
        "book_id": "2",
        "title": "Quantum Dreams: Journey Beyond Reality",
        "author": "Dr. Ivan Sollix",
        "genre": "Science Fiction"
    },
    {
        "book_id": "3",
        "title": "The Shadows of Victorian London",
        "author": "Penelope Gracefield",
        "genre": "Historical Mystery"
    }
]
```
````

### Ask the model to check whether conditions are satisfied

让模型检查条件是否满足。

```python {linenos=true}
text_1 = f"""
Making a cup of tea is easy! First, you need to get some \
water boiling. While that's happening, \
grab a cup and put a tea bag in it. Once the water is \
hot enough, just pour it over the tea bag. \
Let it sit for a bit so the tea can steep. After a \
few minutes, take out the tea bag. If you \
like, you can add some sugar or milk to taste. \
And that's it! You've got yourself a delicious \
cup of tea to enjoy.
"""
prompt = f"""
You will be provided with text delimited by triple quotes. 
If it contains a sequence of instructions, \
re-write those instructions in the following format:

Step 1 - ...
Step 2 - …
…
Step N - …

If the text does not contain a sequence of instructions, \
then simply write \"No steps provided.\"

\"\"\"{text_1}\"\"\"
"""
```

```text {linenos=true}
Step 1 - Get some water boiling.
Step 2 - Grab a cup and put a tea bag in it.
Step 3 - Once the water is hot enough, pour it over the tea bag.
Step 4 - Let it sit for a bit so the tea can steep.
Step 5 - After a few minutes, take out the tea bag.
Step 6 - If you like, add some sugar or milk to taste.
```

```python {linenos=true}
text_2 = f"""
The sun is shining brightly today, and the birds are \
singing. It's a beautiful day to go for a \
walk in the park. The flowers are blooming, and the \
trees are swaying gently in the breeze. People \
are out and about, enjoying the lovely weather. \
Some are having picnics, while others are playing \
games or simply relaxing on the grass. It's a \
perfect day to spend time outdoors and appreciate the \
beauty of nature.
"""
prompt = f"""
You will be provided with text delimited by triple quotes. 
If it contains a sequence of instructions, \
re-write those instructions in the following format:

Step 1 - ...
Step 2 - …
…
Step N - …

If the text does not contain a sequence of instructions, \
then simply write \"No steps provided.\"

\"\"\"{text_2}\"\"\"
"""
```

```text {linenos=true}
No steps provided.
```

### "Few-shot" prompting

给LLM例子，告诉它要如何回答问题。

```python {linenos=true}
prompt = f"""
Your task is to answer in a consistent style.

<child>: Teach me about patience.

<grandparent>: The river that carves the deepest \
valley flows from a modest spring; the \
grandest symphony originates from a single note; \
the most intricate tapestry begins with a solitary thread.

<child>: Teach me about resilience.
"""
```

```text {linenos=true}
<grandparent>: The mighty oak stands tall and unwavering, having grown from a tiny acorn. Every storm that bends its branches only strengthens its roots. Similarly, a pearl forms from persistent grit within an oyster. Resilience is born from facing life's trials and emerging stronger, one step at a time.
```

## Principle 2: Give the model time to “think”

让模型有时间去“思考”。

### Specify the steps required to complete a task.

指出要完成任务的步骤。

```python {linenos=true}
text = f"""
In a charming village, siblings Jack and Jill set out on \
a quest to fetch water from a hilltop \
well. As they climbed, singing joyfully, misfortune \
struck—Jack tripped on a stone and tumbled \
down the hill, with Jill following suit. \
Though slightly battered, the pair returned home to \
comforting embraces. Despite the mishap, \
their adventurous spirits remained undimmed, and they \
continued exploring with delight.
"""

prompt_1 = f"""
Perform the following actions: 
1 - Summarize the following text delimited by triple \
backticks with 1 sentence.
2 - Translate the summary into French.
3 - List each name in the French summary.
4 - Output a json object that contains the following \
keys: french_summary, num_names.

Separate your answers with line breaks.

Text:
```{text}```
"""
```

````text {linenos=true}
1 - In a charming village, siblings Jack and Jill set out on a quest to fetch water from a hilltop well but ended up tumbling down and returned home slightly battered yet undeterred in spirit.

2 - Dans un charmant village, les frères et sœurs Jack et Jill partent en quête d'eau à un puits au sommet d'une colline mais finissent par dégringoler et rentrent chez eux légèrement meurtris mais sans que leur esprit d'aventure ne soit entamé.

3 - Jack, Jill

4 - 
```json
{
  "french_summary": "Dans un charmant village, les frères et sœurs Jack et Jill partent en quête d'eau à un puits au sommet d'une colline mais finissent par dégringoler et rentrent chez eux légèrement meurtris mais sans que leur esprit d'aventure ne soit entamé.",
  "num_names": 2
}
```
````

Ask for output in a specified format.

让LLM生成特定格式的内容。

```python {linenos=true}
text = f"""
In a charming village, siblings Jack and Jill set out on \
a quest to fetch water from a hilltop \
well. As they climbed, singing joyfully, misfortune \
struck—Jack tripped on a stone and tumbled \
down the hill, with Jill following suit. \
Though slightly battered, the pair returned home to \
comforting embraces. Despite the mishap, \
their adventurous spirits remained undimmed, and they \
continued exploring with delight.
"""

prompt_2 = f"""
Your task is to perform the following actions: 
1 - Summarize the following text delimited by 
  <> with 1 sentence.
2 - Translate the summary into French.
3 - List each name in the French summary.
4 - Output a json object that contains the 
  following keys: french_summary, num_names.

Use the following format:
Text: <text to summarize>
Summary: <summary>
Translation: <summary translation>
Names: <list of names in summary>
Output JSON: <json with summary and num_names>

Text: <{text}>
"""
```

```text {linenos=true}
Summary: In a charming village, siblings Jack and Jill had an accident while fetching water but remained adventurous despite their mishap.
Translation: Dans un charmant village, les frères et sœurs Jack et Jill ont eu un accident en allant chercher de l'eau, mais sont restés aventureux malgré leur mésaventure.
Names: [Jack, Jill]
Output JSON: {"french_summary": "Dans un charmant village, les frères et sœurs Jack et Jill ont eu un accident en allant chercher de l'eau, mais sont restés aventureux malgré leur mésaventure.", "num_names": 2}
```

### Instruct the model to work out its own solution before rushing to a conclusion

让模型在给出答案之前说出来它的解决过程。

```python {linenos=true}
prompt = f"""
Determine if the student's solution is correct or not.

Question:
I'm building a solar power installation and I need \
 help working out the financials. 
- Land costs $100 / square foot
- I can buy solar panels for $250 / square foot
- I negotiated a contract for maintenance that will cost \
me a flat $100k per year, and an additional $10 / square \
foot
What is the total cost for the first year of operations 
as a function of the number of square feet.

Student's Solution:
Let x be the size of the installation in square feet.
Costs:
1. Land cost: 100x
2. Solar panel cost: 250x
3. Maintenance cost: 100,000 + 100x
Total cost: 100x + 250x + 100,000 + 100x = 450x + 100,000
"""
```

```text {linenos=true}
The student's solution is very close to correct but has a minor error in the maintenance cost calculation.

Let's break down the costs step by step:

1. **Land cost**: The cost of the land is $100 per square foot. So for \( x \) square feet, the land cost is \( 100x \).

2. **Solar panel cost**: The cost of solar panels is $250 per square foot. So for \( x \) square feet, the solar panel cost is \( 250x \).

3. **Maintenance cost**: The annual maintenance cost is a flat $100,000 plus $10 per square foot. 
   - The correct formula for maintenance is given as $100,000 + 10x$, but the student mistakenly used $100x in their calculations.

Combining these costs:

- Land cost: \( 100x \)
- Solar panel cost: \( 250x \)
- Maintenance cost: \( 100,000 + 10x \)

To find the total cost, add all these components together:

\[
\text{Total cost} = 100x + 250x + 100,000 + 10x = 360x + 100,000
\]

The revised total cost for the first year as a function of the number of square feet \( x \) is: 

\[
\boxed{360x + 100,000}
\]

Thus, the student's solution is not correct. The correct total cost as a function of the number of square feet should be \( 360x + 100,000 \).

```

````python {linenos=true}
prompt = f"""
Your task is to determine if the student's solution \
is correct or not.
To solve the problem do the following:
- First, work out your own solution to the problem including the final total. 
- Then compare your solution to the student's solution \
and evaluate if the student's solution is correct or not. 
Don't decide if the student's solution is correct until 
you have done the problem yourself.

Use the following format:
Question:
```
question here
```
Student's solution:
```
student's solution here
```
Actual solution:
```
steps to work out the solution and your solution here
```
Is the student's solution the same as actual solution \
just calculated:
```
yes or no
```
Student grade:
```
correct or incorrect
```

Question:
```
I'm building a solar power installation and I need help \
working out the financials. 
- Land costs $100 / square foot
- I can buy solar panels for $250 / square foot
- I negotiated a contract for maintenance that will cost \
me a flat $100k per year, and an additional $10 / square \
foot
What is the total cost for the first year of operations \
as a function of the number of square feet.
``` 
Student's solution:
```
Let x be the size of the installation in square feet.
Costs:
1. Land cost: 100x
2. Solar panel cost: 250x
3. Maintenance cost: 100,000 + 100x
Total cost: 100x + 250x + 100,000 + 100x = 450x + 100,000
```
Actual solution:
"""
````

````text {linenos=true}
```
Let \( x \) be the size of the installation in square feet.

Costs:
1. Land cost: \( 100x \)
2. Solar panel cost: \( 250x \)
3. Maintenance cost: \( 100,000 \) (flat fee) + \( 10x \)

Total cost:
\[ 100x + 250x + 100,000 + 10x \]

Combine like terms:
\[ 100x + 250x + 10x + 100,000 = 360x + 100,000 \]

So, the total cost for the first year of operations as a function of the number of square feet is:
\[ 360x + 100,000 \]
```
Is the student's solution the same as the actual solution just calculated:
```
No
```
Student grade:
```
Incorrect
```
````

## Model Limitations: Hallucinations

Boie is a real company, the product name is not real.

```python {linenos=true}
prompt = f"""
Tell me about AeroGlide UltraSlim Smart Toothbrush by Boie
"""
```

```text {linenos=true}
As of my knowledge cutoff date in October 2023, there is no specific product known as the "AeroGlide UltraSlim Smart Toothbrush" by Boie. Boie USA is a company known for its eco-friendly personal care products, including toothbrushes made from antimicrobial materials. Their toothbrushes generally focus on sustainability, with features like replaceable heads and materials designed to reduce bacteria build-up.

It is possible that "AeroGlide UltraSlim Smart Toothbrush" may be a new product released after my last update, or it could be a rebranding of an existing product. To provide accurate and current information, I would recommend visiting the official Boie USA website or contacting their customer service directly for the most recent and detailed information about their product lineup.
```

# Iterative Prompt Development

## Generate a marketing product description from a product fact sheet

````python {linenos=true}
fact_sheet_chair = """
OVERVIEW
- Part of a beautiful family of mid-century inspired office furniture, 
including filing cabinets, desks, bookcases, meeting tables, and more.
- Several options of shell color and base finishes.
- Available with plastic back and front upholstery (SWC-100) 
or full upholstery (SWC-110) in 10 fabric and 6 leather options.
- Base finish options are: stainless steel, matte black, 
gloss white, or chrome.
- Chair is available with or without armrests.
- Suitable for home or business settings.
- Qualified for contract use.

CONSTRUCTION
- 5-wheel plastic coated aluminum base.
- Pneumatic chair adjust for easy raise/lower action.

DIMENSIONS
- WIDTH 53 CM | 20.87”
- DEPTH 51 CM | 20.08”
- HEIGHT 80 CM | 31.50”
- SEAT HEIGHT 44 CM | 17.32”
- SEAT DEPTH 41 CM | 16.14”

OPTIONS
- Soft or hard-floor caster options.
- Two choices of seat foam densities: 
 medium (1.8 lb/ft3) or high (2.8 lb/ft3)
- Armless or 8 position PU armrests 

MATERIALS
SHELL BASE GLIDER
- Cast Aluminum with modified nylon PA6/PA66 coating.
- Shell thickness: 10 mm.
SEAT
- HD36 foam

COUNTRY OF ORIGIN
- Italy
"""
prompt = f"""
Your task is to help a marketing team create a 
description for a retail website of a product based 
on a technical fact sheet.

Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.

Technical specifications: ```{fact_sheet_chair}```
"""
````

````text {linenos=true}
**Product Description:**

Elevate your workspace with the sophisticated charm of our mid-century inspired office chair, expertly crafted to blend style, comfort, and functionality. This chair is a proud member of our exquisite office furniture collection, which includes filing cabinets, desks, bookcases, and meeting tables. 

**Design and Versatility:**
Select from an alluring range of shell colors and base finishes to seamlessly complement any interior décor. Choose between a plastic back with front upholstery (SWC-100) or opt for full upholstery (SWC-110) available in 10 premium fabrics and 6 luxurious leather options. For the base, pick from stainless steel, matte black, gloss white, or chrome finishes, ensuring your chair is a perfect match for your aesthetic preferences. The chair is also available with or without armrests to suit your comfort needs.

**Construction and Comfort:**
Designed for both home and business settings, our chair is robust enough for contract use. It features a sturdy 5-wheel plastic coated aluminum base and a pneumatic adjust system that allows easy height adjustments. The details extend to offering soft or hard-floor caster options and two choices of seat foam densities (medium at 1.8 lb/ft³ and high at 2.8 lb/ft³) for personalized comfort. Armrests come with 8 adjustable positions in PU material for added flexibility.

**Dimensions:**
- Width: 53 cm (20.87”)
- Depth: 51 cm (20.08”)
- Height: 80 cm (31.50”)
- Seat Height: 44 cm (17.32”)
- Seat Depth: 41 cm (16.14”)

**Quality Materials:**
Constructed in Italy, every chair boasts a cast aluminum shell base with modified nylon PA6/PA66 coating, ensuring durability and longevity. The shell is a solid 10 mm thick, providing stability and support. The seat is made of high-quality HD36 foam, guaranteeing comfort even during long hours of use.

Enhance your office with a chair that exemplifies refined elegance and superior craftsmanship. Choose our mid-century inspired office chair and experience the perfect blend of style and functionality.
````

## Issue 1: The text is too long

问题1：文本过长

````python {linenos=true}
prompt = f"""
Your task is to help a marketing team create a 
description for a retail website of a product based 
on a technical fact sheet.

Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.

Use at most 50 words.

Technical specifications: ```{fact_sheet_chair}```
"""
````

````text {linenos=true}
Discover mid-century inspired elegance with our versatile office chair, perfect for home or professional settings. Customize with various shell colors, base finishes, and upholstery options. Ergonomically designed with adjustable height, 5-wheel base, and optional armrests. Crafted in Italy for premium quality and style.
````

## Issue 2. Text focuses on the wrong details

Ask it to focus on the aspects that are relevant to the intended audience.

````python {linenos=true}
prompt = f"""
Your task is to help a marketing team create a 
description for a retail website of a product based 
on a technical fact sheet.

Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.

The description is intended for furniture retailers, 
so should be technical in nature and focus on the 
materials the product is constructed from.

Use at most 50 words.

Technical specifications: ```{fact_sheet_chair}```
"""
````

````text {linenos=true}
Mid-century inspired office chair with plastic coated aluminum base, pneumatic height adjustment, and HD36 foam seat. Available in plastic back/front upholstery or full upholstery in 10 fabrics and 6 leathers. Choice of stainless steel, matte black, gloss white, or chrome base finish. Made in Italy for home or business use.
````

## Issue 3. Description needs a table of dimensions

Ask it to extract information and organize it in a table.

````python {linenos=true}
prompt = f"""
Your task is to help a marketing team create a 
description for a retail website of a product based 
on a technical fact sheet.

Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.

The description is intended for furniture retailers, 
so should be technical in nature and focus on the 
materials the product is constructed from.

At the end of the description, include every 7-character 
Product ID in the technical specification.

After the description, include a table that gives the 
product's dimensions. The table should have two columns.
In the first column include the name of the dimension. 
In the second column include the measurements in inches only.

Give the table the title 'Product Dimensions'.

Format everything as HTML that can be used in a website. 
Place the description in a <div> element.

Technical specifications: ```{fact_sheet_chair}```
"""
````

````text {linenos=true}
```html
<div>
    <h2>Mid-Century Inspired Office Chair</h2>
    <p>Discover the blend of elegance and functionality with our mid-century inspired office chair, meticulously crafted to meet both home and business needs. This chair is a part of an exquisite collection that includes filing cabinets, desks, bookcases, meeting tables, and more. You can customize your seating experience with several options for shell color and base finishes. Choose between a sleek plastic back with front upholstery (SWC-100) or indulge in the comfort of full upholstery (SWC-110), available in a variety of 10 fabric and 6 leather options.</p>
    
    <p>The chair boasts a robust 5-wheel plastic coated aluminum base, designed for durability and ease of movement. The base finishes offered include stainless steel, matte black, gloss white, or chrome, perfectly complementing any decor. The pneumatic chair adjustment mechanism provides smooth raising and lowering actions for ergonomic comfort. Armrest options include an armless version or an 8 position PU armrest, catering to individual preferences.</p>
    
    <p>Constructed in Italy, the chair integrates a cast aluminum shell with a modified nylon PA6/PA66 coating and has a shell thickness of 10 mm. It features a high-density HD36 foam seat, ensuring long-lasting comfort. Select between soft or hard-floor casters to suit your floor type and two seat foam density options, medium (1.8 lb/ft<sup>3</sup>) or high (2.8 lb/ft<sup>3</sup>), to fit your specific seating needs.</p>
    
    <p>This chair is not only stylish but also built to withstand contract use, making it a perfect choice for various settings, from private office spaces to bustling business environments.</p>

    <p>Product IDs: SWC100, SWC110</p>
</div>

<h3>Product Dimensions</h3>
<table border="1">
    <tr>
        <th>Dimension</th>
        <th>Measurement (inches)</th>
    </tr>
    <tr>
        <td>Width</td>
        <td>20.87</td>
    </tr>
    <tr>
        <td>Depth</td>
        <td>20.08</td>
    </tr>
    <tr>
        <td>Height</td>
        <td>31.50</td>
    </tr>
    <tr>
        <td>Seat Height</td>
        <td>17.32</td>
    </tr>
    <tr>
        <td>Seat Depth</td>
        <td>16.14</td>
    </tr>
</table>
```
````