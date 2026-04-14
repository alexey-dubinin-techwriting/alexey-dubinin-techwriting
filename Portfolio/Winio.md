
# **Winio**  
# **Esports Analytics Platform** 

# Product overview

### *ML architecture, data sources, infrastructure, and security*

# Contents

- [Overview](#overview)
- [How the analytics work](#how-the-analytics-work)
- [ML architecture](#ml-architecture)
  - [Three models per match](#three-models-per-match)
  - [Hero embeddings for Dota 2](#hero-embeddings-for-dota-2)
  - [Proprietary rating system](#proprietary-rating-system)
  - [Training dataset scale](#training-dataset-scale)
- [Data](#data)
  - [Historical data](#historical-data)
  - [Live data](#live-data)
- [Platform architecture](#platform-architecture)
  - [Backend and data processing](#backend-and-data-processing)
  - [Frontend and analytics access](#frontend-and-analytics-access)
- [Real-time processing](#real-time-processing)
- [Reliability and scalability](#reliability-and-scalability)
- [Security](#security)
- [Payments and feature access](#payments-and-feature-access)
- [Glossary](#glossary)

# Overview

Winio is an AI-driven esports analytics platform delivering predictions for Dota 2 and CS2. Its proprietary ML models predict a range of match events: match winners, total maps, map outcomes, and many other events specific to each game.

The platform provides users with analytics that are difficult to produce independently: win probabilities calculated from 350,000+ matches, individual player profiles, and real-time draft analysis. Every prediction comes with a live odds aggregator that scans the market: users see the full picture and make their own decisions. Winio operates independently from bookmakers, which ensures objective analytics.

Winio operates across three stages: pre-draft, post-draft, and live during the match. Predictions update automatically at every stage, so users always see the current picture.

This document explains how the platform works: where the data comes from, how the models operate, and what technical decisions drive the stability and accuracy of the analytics.

For definitions of key terms used throughout this document, see the **Glossary**.

# How the analytics work

Winio does not predict exact match outcomes. The platform calculates probabilities, and is transparent about its confidence level.

Each match page has three tabs:

* **Match overview** — each team's win probability as a percentage, alongside the live score.

* **Prediction breakdown** — a written analysis covering what influenced the prediction, how the draft was assessed, and which factors carried the most weight.

* **Odds aggregator** — live odds from across the market, updated every 5 seconds.

Predictions update automatically as new data arrives: first before the draft, then once heroes or maps are known, then continuously during the match. Everything happens on one screen.

# ML architecture

This is the core of the platform.

Winio's proprietary multi-layer ML pipeline generates predictions from real match data collected over the past three years.

## Three models per match

Three models run sequentially for each match.

The **pre-draft model** estimates win probability before heroes or maps are known. It draws on team form, head-to-head history, and player ratings.

The **post-draft model** activates once the draft is complete. It analyzes the specific hero or map composition and recalculates probabilities based on synergies, counterpicks, and map-specific statistics.

The **live model** runs in real time during the match. It factors in the current score and game state, continuously updating the prediction.

## Hero embeddings for Dota 2

To give the model a meaningful understanding of heroes, each of the 163 heroes is assigned a numerical profile called an **embedding**.

An **embedding** is a vector of numbers describing the hero through its actual behavior in matches: role, strength at different game stages, and interactions with allies and enemies.

The key property of embeddings is that heroes with similar roles and playstyles end up close together in numerical space, like points on a map. This lets the model pick up on synergies and counterpicks the way an experienced player would.

For example, the model learned on its own that Io and Tiny amplify each other, and that Broodmother struggles against AoE compositions. This analysis is based on a pattern repeated across hundreds of thousands of matches.

The model trains embeddings on **90,000+ drafts,** and updates them as the meta evolves with new patches and balance changes.

## Proprietary rating system

Winio maintains its own ratings, separately for teams and for players, across more than **8,000 teams** in Dota 2 and CS2.

The key difference from other rating systems is that ratings recalculate after every match, not weekly or monthly. This matters because a team's form can shift over just a few days, and the next match prediction needs to reflect that.

Player ratings account for individual performance adjusted for role and game context. The methodology is proprietary and central to the quality of Winio's predictions. Ratings feed into every prediction on the platform.

## Training dataset scale

| Metric | Value |
| :---- | :---- |
| Dota 2 matches | 210,000 |
| CS2 matches | 140,000 |
| CS2 players with individual profiles | 5,200+ |
| Dota 2 players with individual profiles | 1,100+ |
| Dota 2 heroes with embeddings | 163 |
| Drafts | 90,000+ |
| Unique models (pre-match + live) | 20+ |
| Variables per model | up to 50+ |

# Data

Winio's predictions are built on two types of data.

## Historical data

Over three years, the team collected and structured match history across Dota 2 and CS2, covering all patches and all major tournaments. A proprietary parser processes match replay files and extracts data in a structured format: picks and bans, items purchased by the second, kill counts, etc.

Most importantly, the parser captures XYZ coordinates for every hero at every second of the match.

The model sees the full flow of the game: where heroes moved, when they used a smoke, and how ganks developed.

For example, if the model sees five heroes converging on one point, it recognizes that a gank is going to happen, and can assess how that is likely to affect the rest of the match.

## Live data

Live match data comes from official providers of tournament organizers, like PGL and ESL.

It flows over a WebSocket **at broadcast speed**, with the same latency as a Twitch stream. The live model immediately sees and factors in everything happening in the game.

> **Note:** Winio builds predictions from its own data on teams, players, and matches. Bookmaker odds are not used as input to the model.

# Platform architecture

The platform consists of a microservices architecture: each component handles a specific part of the logic and can be updated independently of the others. This provides flexibility under load and resilience to individual service failures.

## Backend and data processing

The system processes data asynchronously, and does not block during peak traffic. Queues and caching smooth out load spikes, allowing data processing to unfold in stages, without degrading performance.

## Frontend and analytics access

The frontend uses server-side rendering (SSR).

The server assembles pages and sends them ready to display, reducing load times and dependency on the user's device. The interface is responsive across desktop and mobile.

# Real-time processing

Winio processes in-game events in streaming mode, factoring data into calculations as soon as it arrives. When inputs change, predictions recalculate automatically.

The system updates bookmaker odds on each match page every **5 seconds**.

# Reliability and scalability

Services run in containers, which simplifies scaling and updating individual components without taking the platform offline. Automatic restarts recover from failures. Regular backups reduce the risk of data loss.

The platform continuously monitors infrastructure and surfaces problems before they affect users. A separate data quality layer runs automatically, detecting anomalies and inconsistencies that could compromise prediction accuracy.

# Security

The platform uses authentication and role-based access control tied to subscription levels. User sessions are protected by modern token management. Users verify their accounts to reduce the risk of unauthorized access.

# Payments and feature access

Payment logic is decoupled from the analytics components. Winio processes subscriptions through external payment services. Once a payment is confirmed, the system automatically grants the corresponding level of access.

For details, see the Subscription page.

# Glossary

**Asynchronous processing** — a method of handling tasks in parallel without waiting for each to complete before starting the next. It keeps the system responsive under high load.

**Container** — an isolated runtime environment in which a single platform service runs. Containers allow components to be updated and scaled independently.

**Counterpick** — a hero selection that has a clear advantage over a specific enemy hero based on abilities or characteristics.

**Draft** — the pre-match phase in which teams select heroes (Dota 2) or maps (CS2), including picks and bans. The draft composition directly influences team strategy and win probability.

**Embedding** — a numerical representation of an object (hero, player, map) as a vector. The model uses embeddings to understand relationships between objects: synergies, counterpicks, and roles.

**Live model** — the ML model that updates predictions in real time during a match, based on the current score and game state.

**Microservices architecture** — a system design approach implementing each function as an independent service with its own logic and data store.

**ML pipeline** — the sequence of data processing steps and model applications, from raw data ingestion to generating a final prediction.

**Post-draft model** — the ML model that recalculates win probability after the draft is complete, accounting for the specific composition, synergies, and counterpicks.

**Pre-draft model** — the ML model that estimates win probability before heroes or maps are known. It relies on team form and historical data.

**Queues** — a mechanism where tasks are lined up and processed in order. It smooths out traffic spikes and prevents system overload.

**Parser** — a program that reads data from a file or stream, and converts it into a structured format suitable for model processing.

**Rating system** — Winio's proprietary system for evaluating team and player level. It recalculates after every match, and feeds into all prediction models as an input factor.

**SSR (server-side rendering)** — a technique where HTML pages are assembled on the server and sent to the user ready to display. Reduces load times and dependency on device performance.

**Synergy** — the mutual amplification of heroes on the same team. The abilities or characteristics of one hero enhance the effectiveness of another.

**Token (auth)** — an encrypted string that confirms a user's identity and access rights. This is not related to cryptocurrency tokens.

**WebSocket** — a data transfer protocol that maintains a continuous stream of information between server and client, eliminating the need for a separate request on every update.
