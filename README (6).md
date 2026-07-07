# Maritime Cargo-Flow Chatbot

A natural-language interface for maritime tanker cargo data. Ask a question in plain English, like "MR tanker gasoline exports from the US Gulf in June 2026, grouped by destination," and get a real, grounded answer back.

This public version runs entirely on synthetic (fake) data, so it needs no API key and no special access. The production version connects to a licensed maritime data API, which is why real figures are not shown here. The architecture is identical in both; only the data source changes.

## The core idea: the model never touches the numbers

The language model is not the chatbot. It is the part that handles words. It sits on either side of the real work as a translator and a writer, and the actual data is owned entirely by ordinary Python code, which is the source of truth.

There are three stages to every question.

**Stage one, translate.** The model reads your plain-English question and converts it into a structured query: activity (export or import), origin, destination, product, vessel class, and date range. It fills a schema rather than free-typing an answer. This is called tool calling.

**Stage two, query.** The Python code takes those parameters, resolves them, and pulls the actual figures from the data source. The model has no say in this step. Every number comes from here.

**Stage three, summarise.** The real figures are handed back to the model, which writes an analyst-style summary using only what the query returned. It is instructed never to invent a figure.

That separation is the whole point. If you let a language model make up cargo statistics, it will, confidently and wrongly. By keeping it to translating and narrating while grounded code owns the data and the math, every number stays real and traceable.

## Two tools, and the model chooses between them

The chatbot has two capabilities, and the model decides which one fits each question.

**Cargo flows** answers single-period totals. "How much moved from A to B in month X," with an optional breakdown by destination or origin.

**Cargo time series** answers trends over time. "How has X changed month by month," returning a series of buckets rather than one total.

You can watch the routing happen. Before each answer, the notebook prints which tool the model picked and the exact structured query it produced. That printout is the guardrail: it shows you the model is genuinely translating the question, not guessing, and it lets you catch a mistranslation immediately.

## How it runs

The language model is a small open-source model (Qwen2.5) running on a cloud GPU through a notebook. Nothing about the design is tied to that specific model. The engine was swapped several times during development, and the translate, query, summarise core never changed. Good architecture survives swapping its components.

To run this public version:

1. Open the notebook in a cloud notebook environment with a GPU enabled.
2. Run the cells in order, top to bottom, waiting for each to finish before starting the next.
3. When the chat prompt appears, type a question and press Enter.

No API key is required. All data is synthetic and clearly labelled as such.

## Honest limitations

**Small-model translation is imperfect.** The model occasionally misreads a question, for example inventing a value that was never asked for, or picking the wrong tool between the two similar cargo tools. The printed query line exists so these slips are visible rather than hidden. This is a deliberate tradeoff for using a free, self-hostable model rather than a large hosted one.

**One time window per query.** Each tool handles a single date range. A question that compares two periods, like "May versus June," cannot be answered in a single call and needs to be asked as two separate questions.

**Volumes only, in the public version.** The tools answer questions about cargo volumes moved. They do not cover freight rates, prices, or vessel positions. A freight-pricing tool was built and routes correctly, but it depends on a data entitlement not included here, so it is left out of this version.

**Synthetic data.** Every figure in this public version is randomly generated for demonstration and does not represent real trade flows.

## Why it is built this way

The design goal was a language interface that is genuinely trustworthy over real data: one that never fabricates a number, that shows its working, and that admits what it cannot do. The separation of concerns (model for language, code for data) is what makes that possible, and it is the part worth reusing for any grounded language tool over a real dataset.
