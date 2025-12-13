# AmazonConnect-AI-Agents-ListSpansViewer
Web Based Viewer for ListSpansAPI
# ListSpans Viewer for Amazon Connect AI Agents

[![AWS CloudFormation](https://img.shields.io/badge/AWS-CloudFormation-orange?logo=amazon-aws)](https://aws.amazon.com/cloudformation/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-5.1.0-blue.svg)](https://github.com/yourusername/listspans-viewer/releases)

A web-based visualization tool for the Amazon Connect AI Agents [ListSpans API](https://docs.aws.amazon.com/amazon-q-connect/latest/APIReference/API_ListSpans.html). Deploy with a single CloudFormation template and instantly start debugging and analyzing your AI agent sessions.

![ListSpans Viewer Screenshot](docs/screenshot.png)

## 🎯 What is This?

When building AI agents with Amazon Connect, debugging agent behavior can be challenging. The ListSpans API provides detailed telemetry about what happened during an agent session, including:

- **Inference calls** - LLM invocations with input/output messages
- **Tool executions** - Which tools were called and their results
- **Agent invocations** - Sub-agent calls and orchestration
- **Token usage** - Input/output token counts per call
- **Timing data** - Duration of each operation
- **Error details** - What went wrong and where

This tool provides a clean, visual interface to explore this data without writing code or using the AWS CLI.

## ✨ Features

- 🚀 **One-Click Deploy** - Single CloudFormation template deploys everything
- 🔐 **Secure** - Uses IAM roles, no credentials in the browser
- 🎨 **Visual Interface** - Color-coded spans, expandable details, syntax highlighting
- 🔍 **Filtering** - Filter by span type (inference, tool, agent) or errors
- 📊 **Statistics** - Token counts, success/error rates at a glance
- 📋 **Export** - Download raw JSON or copy to clipboard
- 🌐 **Multi-Region** - Query any supported AWS region
- ⚡ **Serverless** - No servers to manage, scales automatically

## 🏗️ Architecture
