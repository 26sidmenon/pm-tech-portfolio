# Kalki WhatsApp Chatbot & Website Integration

## Project Overview
Built and deployed a RAG-based AI chatbot on a Wix website, 
trained on proprietary product documentation, with WhatsApp 
Business integration for automated lead capture — all without 
any engineering support.

## Problem Statement
- Website visitors had no way to get instant answers about Kalki
- Incoming WhatsApp messages were being missed due to inconvenient interface
- No automated way to capture and qualify leads

## Solution
Designed and deployed an end-to-end customer engagement stack:
1. AI chatbot trained on 45-page Kalki product manual
2. WhatsApp Business integration for direct lead capture
3. Pre-filled WhatsApp message for purchase inquiries

## Tools Used
- **Chatbase** - RAG-based chatbot platform
- **WhatsApp Business** - Lead capture and customer communication
- **Wix** - Website platform
- **Claude AI** - Generated comprehensive FAQ from product manual

## What I Learned
- What RAG (Retrieval-Augmented Generation) is and how it works
- Difference between a reactive chatbot and agentic AI
- How to embed third-party tools into a Wix website via custom code
- WhatsApp Business API limitations and workarounds
- How Meta controls WhatsApp API access

## Key Decisions Made
| Decision | Options Considered | Choice | Reason |
|----------|-------------------|--------|--------|
| Chatbot platform | Wix AI Chat, Chatbase, Botpress | Chatbase | Better reliability, easy document upload |
| WhatsApp integration | Wix native, WhatsApp Business API | WhatsApp button + pre-filled message | Free, instant, no API approval needed |
| Knowledge base | Manual FAQ writing, PDF upload | AI-generated FAQ from manual | Faster, more comprehensive coverage |

## Outcome
- 24/7 AI-powered customer engagement live on website
- Visitors can get instant answers about Kalki at any time
- Purchase inquiries land directly in WhatsApp inbox
- Zero ongoing maintenance required for basic operation

## What I Would Do Next
- Upgrade to WhatsApp Business API for full chatbot integration
- Add lead capture (name + email) before chat begins
- Track chatbot conversations to identify knowledge gaps
- Make it more agentic by auto-logging leads to a spreadsheet

## Concepts Learned
- **RAG (Retrieval-Augmented Generation)**: AI that retrieves 
  relevant information from a knowledge base before generating answers
- **Agentic AI**: AI that takes autonomous actions across multiple 
  steps toward a goal, beyond just answering questions
- **WhatsApp Business API**: Meta-controlled API for programmatic 
  WhatsApp messaging, requires approval and has usage limits
```
