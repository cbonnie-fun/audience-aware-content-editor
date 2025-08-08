# Create an audience-aware technical content editor with Gemini

Keeping the target audience in mind is crucial for creating effective and usable technical documentation. This guide describes how to create an audience-aware content editor that adapts technical content to different user groups, so you can make your developer documentation more user focused, engaging, and helpful.

In this guide, you’ll build a content editor by:

* Clearly defining user personas
* Writing an effective prompt
* Running the prompt by using the Gemini command line interface (CLI)

This guide is intended for technical writers and other content authors who want to improve the usability of their content. All the steps in this guide include sample code or content that you can use to complete the step.

## Before you begin

* This guide assumes that you’re familiar with using CLI tools and writing prompts for Large Language Models (LLMs). If not, review the following resources before using this guide:

    * [What are LLMs?](https://www.ibm.com/think/topics/large-language-models)
    * [Introduction to prompting](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/prompts/introduction-prompt-design)
    * [CLI basics for developers](https://daily.dev/blog/cli-basics-for-developers)

* If you haven’t already, [install](https://github.com/google-gemini/gemini-cli) and [set up authentication](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/authentication.md) with the Gemini CLI.

# Step 1: Define user personas

Using [user personas](https://www.nngroup.com/articles/persona/) is a prompting technique that lets you specify the expected output of a prompt by providing important context to a model about the intended user, such as the user’s background, goals, and needs.

Review the following example personas of a junior developer, a senior developer, and an engineering lead. You’ll refer to these personas in the next step, where you’ll create a prompt.

* Junior developer
  * Experience: 0-2 years.
  * User goal: To complete a task successfully and learn the fundamentals.
  * Content needs: Clear, step-by-step instructions. Definitions of key terms and concepts. Code samples with detailed explanations. A supportive and encouraging tone.
* Senior developer
  * Experience: 5+ years.
  * User goal: To quickly understand the purpose behind technical decisions and how to integrate new technologies.
  * Content needs: Concise, to-the point information. Best practices, performance, and other advanced concepts. Assumed to have a strong foundational knowledge of most computer science and engineering topics.
* Engineering lead
  * Experience: Varies, often 7+ years.
  * User goal: To make informed decisions about technology adoption and project planning.
  * Content needs: High-level overviews. Information about scalability, security, and business impact. How a technology will affect team workflows and resources.

## Step 2: Write a prompt

1. In your text editor of choice, create a new file and name it `content-editor-template.txt`.
1. In `content-editor-template.txt`, write a prompt that includes the following elements:

   * **A role assignment**: Tells Gemini to adopt a specific persona that activates its knowledge about good technical writing practices.
   * **A core task**: Tells Gemini the high-level objective, which is rewriting content for specific audiences.
   * **Instructions**: Translates the core task into a precise and repeatable procedure. Provides step-by-step rules for Gemini to follow, using the following model:

        * **IF** `Audience` = `Junior Developer`, **THEN** execute Rules for Junior Developer
        * **IF** `Audience` = `Senior Developer`, **THEN** execute Rules for Senior Developer
        * **IF** `Audience` = `Engineering Lead`, **THEN** execute Rules for Engineering Lead

   The sets of rules are written based on the implied needs of each defined user persona.

   * **Variable placeholders**: Prepares Gemini to receive input and target audience information so it can complete its task.

   * **An output constraint**: Tells Gemini to skip any conversational filler in the output and provide only final text.

   For example:

    ```
    You are an expert technical writer with a talent for adapting complex topics for different audiences. Your primary goal is to rewrite the input content while preserving its core technical accuracy.

    ## Input Content
    {{CONTENT}}

    ## Target Audience
    {{AUDIENCE}}

    ## Instructions
    Rewrite the input content according to the rules for the specified target audience.

    ### Rules for Junior Developer
    * **Tone**: Supportive and encouraging.
    * **Explain**: Define all jargon and abstract concepts using simple terms or analogies.
    * **Structure**: Use a step-by-step format if the topic involves a process.
    * **Code**: Provide a minimal, runnable code snippet with clear comments explaining each line or block.

    ### Rules for Senior Developer
    * **Tone**: Direct, concise, and professional.
    * **Focus**: Emphasize technical best practices, performance trade-offs, and potential edge cases or "gotchas."
    * **Assume**: The reader has a strong command of fundamental concepts and programming patterns.
    * **Compare**: Briefly mention how this approach compares to common alternatives.

    ### Rules for Engineering Lead
    * **Tone**: Strategic and business-oriented.
    * **Summarize**: Begin with a high-level summary (1-2 sentences) of the technology and its value proposition.
    * **Impact**: Focus on the strategic implications for the team and business, such as scalability, security, cost (TCO), and required team skill sets.
    * **Avoid**: Deeply technical implementation details; focus on the "what" and "why," not the "how."

    Output only the final, rewritten content.
    ```

1. Save `content-editor-template.txt` to your `/home` directory.

## Step 3: Run the content editor

1. In your text editor, create a new file and name it `example-content.txt`. This file will contain the technical content you want to rewrite.

1. Copy and paste the following example input content into `example-content.txt`:

```
Our new service implements a Redis-based write-through caching strategy to reduce database latency. All read requests for user profiles first check the Redis cache. If the data is present (a cache hit), it's returned directly. If it's a cache miss, the request queries the primary PostgreSQL database, and the result is asynchronously populated back into the cache before being sent to the client. This minimizes read contention on the main DB.
```

1. Save `example-content.txt` to your `/home` directory.

1. In your terminal, run the following command from your `/home` directory to combine your files and send them to Gemini:

```
cat content-editor-template.txt | sed "s/{{AUDIENCE}}/Engineering Lead/" | sed "s|{{CONTENT}}|$(cat example-content.txt)|" | gemini gen
```

This command performs the following operations:
* Reads the content-editor-template.txt file
* Replaces the {{AUDIENCE}} placeholder with Engineering Lead
* Replaces the {{CONTENT}} placeholder with the example-content.txt file
* Pipes the final, complete prompt to the Gemini CLI

The response contains the revised content. For example:

```
We've implemented a Redis-based caching layer to significantly improve read performance and reduce load on our primary database. This enhancement increases our system's scalability and responsiveness, directly improving user experience while helping manage our total cost of ownership (TCO) by deferring the need for more expensive database scaling solutions. The required team skill set is minimal, aligning with our goal of building resilient and cost-effective systems.
```