# hypertext markup language Text Processor Abstract Model (`website.html.text.processor`)

**Transport name:** `website.html.text.processor`  
**Storage name:** `website_html_text_processor`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `website`

Description: HTML Text Processor Abstract Model

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_with_processing_context` | internal rule | self, IrQweb, cta_data, text_generation_target_lang, text_must_be_translated_for_openai | `website` | model | Initialize HTML processing context similar to website.with_context() pattern :param IrQweb: QWeb rendering environment :param cta_data: Call-to-action data for rendering :param text_generation_target_lang: Target language code :param text_must_be_translated_for_openai: Whether translation is required :return: WebsiteHTMLTextProcessor instance with processing context |
| `_get_processing_cache` | preparation rule | self, cache_key | `website` |  | Get cached data from context, similar to website._get_cached() pattern :param cache_key: Key to get cached data from context :return: Cached data from context :rtype: dict |
| `_update_processing_cache` | internal rule | self, cache_key, updates | `website` |  | Update cached data in context, returning new context :param cache_key: Key to update cached data in context :param updates: Updates to add to cached data :return: WebsiteHTMLTextProcessor instance with updated context |
| `_get_snippet_content` | preparation rule | self, snippet_key | `website` | model | Get the content of a list of snippets as a list of placeholders and their translations :param snippet_key: Snippet key :return: (updated_processor (self), generated_content, translated_content) :rtype: tuple |
| `_get_rendered_snippets_content` | preparation rule | self, snippets | `website` | model | Process rendered snippets :param snippets: Dict of snippet_id keys mapping to the snippet HTML string and the English snippet HTML string :type snippets: dict :return: (updated_processor, generated_content, translated_content) :rtype: tuple |
| `_process_snippet` | background operation | self, snippet, snippet_en | `website` |  | Process a snippet and its translation :param snippet: Snippet HTML string :param snippet_en: English snippet HTML string :type snippet_en: str :return: (updated_processor, placeholders) :rtype: tuple |
| `_render_snippet` | internal rule | self, snippet_key | `website` |  | Handle the rendering of a snippet and its translation :param snippet_key: Snippet key :return: (updated_processor, render, placeholders) :rtype: tuple |
| `_calculate_translation_ratio` | internal rule | self, generated_content, translated_content | `website` | model | Calculate the translation ratio between generated and translated content. :param generated_content: Generated content :param translated_content: Translated content :return: Translation ratio :rtype: float |
| `_update_snippet_content` | internal rule | self, generated_content, snippet_key, snippet_html | `website` | model | Update the content of a snippet with the generated content :param generated_content: Generated content to update :type generated_content: dict :param snippet_key: Snippet key :type snippet_key: str :param snippet_html: Snippet HTML string :type snippet_html: str :return: Updated HTML element with snippet data :rtype: lxml.html.HtmlElement |
| `_compute_placeholder` | computation | self, html_string | `website` |  | Transforms an HTML string by converting specific HTML tags into a custom pseudo-markdown format using context-stored state. :param html_string: The input HTML string to be transformed. :type html_string: str :return: (updated_processor, transformed_string) - The updated processor instance         and the transformed string with HTML tags replaced by pseudo-markdown. :rtype: tuple |
| `_format_replacement` | internal rule | self, html_string, generated_content | `website` |  | Reapplies original HTML formatting by replacing pseudo-markdown with corresponding HTML tags using context-stored state. :param html_string: The source HTML whose replacement has to     receive original formatting :type html_string: str :param generated_content: Dictionary mapping placeholders to generated content :type generated_content: dict :return: The text with HTML tags re-applied. :rtype: str |

Machine-readable definition: `../../../schemas/data/entities/website.html.text.processor.json`.
