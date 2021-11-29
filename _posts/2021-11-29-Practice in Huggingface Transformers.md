---
title: Practice in Huggingface Transformers
key: 202111290
tags: Tokenizer, PLMs
---

# Practice in Huggingface Transformers

This post is just a note summarizing my practice on Pre-trained Language Models (PLMs) using Huggingface Transformers Package. I believe that these points I am summarizing is also confusing to other rookies of coding for transformer-based NLP models.



<!--more-->



## Tokenization



### API Reference

- input format: input dictionary with keys consisting of `input_ids`, `attention_mask`, `token_type_ids`, `labels`.
- text modeling: `str`, `List[str]`(pre-tokenized sequence, aka whitespace-tokenized sequence) , `List[int]`(after invoking `tokenizer.encode` and requiring padding or truncation).
- batched text modeling: add a batch dimension for the above format, and the ambiguity of  `List[str]` will be addressed by the argument `is_split_into_words = False/True`.
- build input: from version `4.4.2`, `tokenizer.__call__` can handle all the situations, including a single text, a single pair of texts, a batch of texts and a batch of pairs of texts.
- what is `token_type_ids`
  - It is a mask to reflect that the current token belongs to which input sequence.
  - only when `return_token_type_ids = True` can we make our tokenization clear and useful.
  - although `tokenizer.encode` can be fed `text` and `text_pair`, it does not have a parameter dubbed as `return_token_type_ids`. It can be only considered as a in-method-concatenation plus tokenization.
  - this field is model-dependent and this dependency is consistent with the corresponding pre-training strategy of original papers.
    - `BERT` is pretrained on MLM and NSP/SOP and can handle the `token_type_ids` input.
    - `GPT2` is pretrained on autoregressive generation and can be fine-tuned on conditional generation, which demand it to handle the `token_type_ids` input.
    - `BART` is pretrained on MLM using the Seq2Seq modeling and targets conditional generation only, which means it cannot handle the `token_type_ids` input. Moreover, `BartTokenizer` does not return valid `token_type_ids` when fed into a text pair.
    - `others` TO-BE-Done



### Special Tokens in PLMs

- `add_special_tokens` in `tokenizer.__call__` is set to `True` as default.
- there two groups of special tokens, comprising U-styled for NLU and G-styled for NLG.
  - U-styled: `cls_token`, `sep_token` and `pad_token` for marking the start, end and padding of a sequence. when `text_pair` is fed into the tokenizer, `sep_token` will be appended into the end of each sequence.
  - G-styled: `bos_token`, `eos_token` and `pad_token` for the same purpose as above. when `text_pair` is fed into the tokenizer, it is usually be accompanied by a `prompt` or `prefix` to perform fine-tuning of conditional generation. At that time, `eos_token` is used to mark the final stop of the whole sequence rather than the end of either.
  - For encoder-decoder architecture, G-styled is the major style. `bos_token` and `eos_token` can also act as `cls_token` and `sep_token`, respectively.
- `BERT`: U-styled - `[CLS]`, `[SEP]`, `[PAD]`
- `GPT2`: G-styled - `<|endoftext|>`, `<|endoftext|>` without primitive padding token
- `BART`: G-styled - `<s>`, `</s>`, `<pad>`
- `others`: TO-BE-Done

