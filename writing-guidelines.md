# Writing Guidelines

1. Be explicit - explicitly state whether a user is in a code editor, terminal, or browser, and when moving between them. Be clear about where lines of code should be added.
2. Be inclusive - words like "simple", "straightforward", or "just" are not a universal truth so avoid them. Use gender-neutral terms when referring to a user. 
3. Be concise - always consider how to make sentences shorter and clearer. Also only focus on the parts that matter to your project - remove unnecessary steps like styling, robust error handling, or polish, unless relevant to your topic.
4. The what and why - our job to help developers become better developers. As well as explain what to do in a given instruction, why you're doing it, and why it works. 
5. Write readable code - use clear variable names, focus on readability and clarity over syntactic sugar or optimizations.

## Structuring Guidelines

1. Always start the article with configuration, project setup, and dependency installations. Do these for your whole project upfront, even if you won't immediately use certain dependencies. This will ensure that the reader is prepared to follow along.
2. Describe a step only when the reader has completed any necessary pre-steps. For example, don't call or use functions that haven't yet been written. 
3. Break large blocks of code into smaller steps that can be explained in isolation.
4. In tutorials, "time to first win" is something we consider. Get to the point of testing with some form of verifiable success as quickly as possible. These quick small wins motivate the reader to continue. 
5. After the build, show a user how to run the project and verify it's working. If there are common gotchas or bugs, please also include how to troubleshoot.
6. In the summary, highlight the key skills the user has demonstrated. Tell them what they should do next - is it to check out a specific docs page? Is it to sign up for Directus Cloud? Watch a video on YouTube? 

## Styling Guidelines

1. You can use first-person plural (we, our) or second-person (you, yours), but do not use first-person singular pronouns (I, mine). Whichever you choose - remain consistent throughout. 
2. We like an oxford comma - add a comma after the penultimate item in a list of three or more.
   - Incorrect Example: I love my parents, my dog and my cat.
   - Correct Example: I love my parents, my dog, and my cat. (notice the comma after dog)
3. Use title case for all headings.
4. All sections of your post should use a Heading 2, and you may also use Heading 3 for sub-sections.
5. Do not format text unless it follows these rules:
   - File and folder names should be _italicized_. 
   - Package names, variables names, and all in-line code should be inside `single back-ticks`. 
   - When describing UI labels, headings, buttons, or menu items on a website, these should be **Bolded and Properly Capitalized**.

### Code Samples

In-line code should be in `single back-ticks`. Use markdown formatting for larger code blocks - a line of triple backticks above and below the snippet. You can specify a language after the opening backticks: 

````
```js
console.log('Hello World');
```
````

You can add colored diffs in your code blocks by using `// [!code ++]` or // `[!code --]` at the end of lines to be highlighted. The comments will not be rendered. For example:

````
```js
console.log('Hello World'); // [!code --]
console.log('Hello, World'); // [!code ++]
```
````

### Images

Make sure images are in your post's directory.

All images must have alt text. Alt text is a brief description of what is happening in an image, used by screen readers for those with vision difficulties as well as search engines. Consider what is important to highlight to a user who may not be able to see the graphic. 

Screenshots should be captured in the highest-quality possible, as images (by default) are displayed in articles at the full width of the article.

If images are not screenshots, please make sure we have the right to reuse and publish them. Try to only include them if they add value.

### Info Boxes

Info boxes can be added to provide additional context that enrich the reader's knowledge. Use a code block with the below markdown syntax to add an info box. Info Boxes are not required to have a successful article.

```
:::info Box title

You can write markdown in the box. Make sure there's a space above and below paragraphs. These boxes [support links](https://directus.io).

:::
```
