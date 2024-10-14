<script>
  import axios from "axios";
  import RecursiveCategory from "./components/RecursiveCategory.svelte";

  let userInput = "";
  let apiResponse = "";
  let isLoading = false;
  let categoryData = null;

  const OPENAI_API_KEY = import.meta.env.VITE_OPENAI_API_KEY;
  async function callAIModel() {
    isLoading = true;
    apiResponse = "";
    categoryData = null;

    console.log("Calling AI model...");

    try {
      const response = await axios.post(
        "https://api.openai.com/v1/chat/completions",
        {
          model: "gpt-3.5-turbo",
          messages: [
            {
              role: "system",
              content:
                "You are a helpful assistant that generates detailed categories and subcategories based on user input.",
            },
            {
              role: "user",
              content: `Based on the following request: "${userInput}", generate a hierarchical list of categories and subcategories. 
              - Provide at least 6-7 top-level categories.
              - For each top-level category, provide subcategories.
              - If a subcategory can be further divided, provide additional subcategories.
              - The output should be in JSON format with each category containing subcategories, and subcategories containing further subcategories if they exist.
              
              Example format:
              {
                "category": "Frontend Development",
                "subcategories": [
                  {
                    "subcategory": "Languages",
                    "subcategories": ["JavaScript", "HTML", "CSS"]
                  },
                  {
                    "subcategory": "Frameworks",
                    "subcategories": ["React", "Vue", "Angular"]
                  }
                ]
              }`,
            },
          ],
          max_tokens: 500,
        },
        {
          headers: {
            Authorization: `Bearer ${OPENAI_API_KEY}`,
            "Content-Type": "application/json",
          },
        }
      );

      const data = response.data;

      if (!data.choices || !data.choices[0]) {
        throw new Error("Invalid response from AI model.");
      }

      let responseText = data.choices[0].message.content.trim();
      console.log("Raw response text:", responseText);

      function cleanResponseText(text) {
        let cleanedText = text
          .replace(/```[\s\S]*?```/g, "")
          .replace(/`/g, "")
          .replace(/\s+/g, " ")
          .trim();

        cleanedText = cleanedText.replace(/,\s*([\]}])/g, "$1");

        let openSquareBrackets = (cleanedText.match(/\[/g) || []).length;
        let closeSquareBrackets = (cleanedText.match(/\]/g) || []).length;
        let openCurlyBrackets = (cleanedText.match(/{/g) || []).length;
        let closeCurlyBrackets = (cleanedText.match(/}/g) || []).length;

        while (closeSquareBrackets < openSquareBrackets) {
          cleanedText += "]";
          closeSquareBrackets++;
        }

        while (closeCurlyBrackets < openCurlyBrackets) {
          cleanedText += "}";
          closeCurlyBrackets++;
        }

        try {
          JSON.parse(cleanedText);
        } catch (error) {
          console.error("Error parsing JSON:", error.message);
          return "{}";
        }

        return cleanedText;
      }

      responseText = cleanResponseText(responseText);
      console.log("Cleaned response text:", responseText);

      try {
        categoryData = JSON.parse(responseText);
        console.log("Parsed category data:", categoryData);
      } catch (error) {
        console.error("Error parsing AI response:", error);
        apiResponse = "Sorry, the response could not be processed.";
      }
    } catch (error) {
      console.error("Error calling AI API:", error);
      apiResponse = error.message || "Sorry, something went wrong.";
    } finally {
      isLoading = false;
    }
  }
</script>

<div class="min-h-screen bg-gray-900 flex flex-col items-center">
  <div class="w-full max-w-md mt-10 px-4">
    <input
      type="text"
      bind:value={userInput}
      placeholder="Enter your request here..."
      class="w-full p-4 text-lg bg-gray-800/50 text-white placeholder-gray-400
             rounded-lg border border-gray-700 shadow-xl
             backdrop-blur-md focus:outline-none
             focus:ring-4 focus:ring-gray-400/20 focus:border-gray-500"
    />
  </div>

  <div class="w-full max-w-md mt-4 px-4">
    <button
      on:click={callAIModel}
      class="w-full p-4 text-lg bg-blue-600 text-white rounded-lg shadow-lg
             hover:bg-blue-700 focus:outline-none focus:ring-4 focus:ring-blue-400"
      disabled={isLoading || !userInput}
    >
      {isLoading ? "Loading..." : "Generate Categories"}
    </button>
  </div>

  {#if categoryData}
    <div class="w-full max-w-3xl mt-8 px-4">
      <h2 class="text-center text-white/80 text-2xl mb-4">
        Generated Category
      </h2>

      <div class="mb-6">
        <h3 class="text-white text-xl font-bold">{categoryData.category}</h3>

        {#each categoryData.subcategories as subcategory}
          {#if subcategory.subcategories}
            <RecursiveCategory {subcategory} />
          {:else}
            <li class="text-white/70">{subcategory}</li>
          {/if}
        {/each}
      </div>
    </div>
  {/if}

  {#if apiResponse}
    <div class="w-full max-w-3xl mt-8 px-4">
      <p class="text-center text-red-500 text-xl">
        {apiResponse}
      </p>
    </div>
  {/if}
</div>
