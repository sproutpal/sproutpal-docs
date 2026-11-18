# Product concept

SproutPal has two parts: a gardening knowledge base for anyone interested in gardening, and garden planning that applies that knowledge to the user's own plot.

See [vision.md](vision.md) for the problem, the target users and the guiding principles.

## Knowledge base

The knowledge base has two parts with different purposes.

### Course

The course covers the **fundamentals** — for fruit trees, for example: planting, pruning, crown forms, rootstocks and pollination.

- It is made of **topics**. Each topic is a lesson, and the course is the topics arranged in a logical learning order.
- Following the course is optional: users can jump to any topic at any time.
- Topics refer to earlier, more basic topics, so users can move back and forth between them.
- Topics are free-form: any number of paragraphs, images and highlighted notes or warnings.

The course is the entry point for beginners who don't yet know what to ask.

### Encyclopedia

The encyclopedia holds the **concrete descriptions**: species, varieties, rootstocks, pests and diseases.

- Users look entries up whenever they need them; they are not part of the course order.
- Entries of the same kind share the same fields — for a fruit variety, for example: ripening time, taste, yield, frost hardiness, pollination, suitable rootstocks and disease resistance. The possible values can differ by species (apple rootstocks are not cherry rootstocks).
- Entries link to each other and to the course: a variety lists its rootstocks and pollinators, and refers to the *Rootstocks* and *Pollination* topics.

### Reliability

- The fundamentals are based on published, reviewed horticultural textbooks, rewritten in our own words.
- Mistakes can happen in any source, so every topic and entry has a visible **change log**.
- Every topic and entry shows the date of its last change.
- Signed-in users are notified when a topic they have already read has changed since.

### Accounts

- Everything can be read **without signing in**, including the course: users can go through the topics in order, but their progress is not saved.
- Users **sign in with an external provider** (single sign-on, for example Google); there is no separate registration.
- Signing in adds:
  - **course progress**, saved across devices;
  - **change notifications** for topics the user has already read.

## Garden planning

| Module | Purpose |
| --- | --- |
| **Plot editor** | Draw the plot as a polygon in meters or on a satellite image, with its fixed elements: house, trees, fences, paving, water taps. |
| **Sun map** | Seasonal sunlight from orientation, latitude and shading objects. |
| **Soil and zones** | Soil type, pH, moisture and drainage per zone. |
| **Planting planner** | Place plants and get conflict warnings based on knowledge base data: light, soil, spacing, pollination, bad neighbors. |
| **Recommender** | "What, where and why": filter plants by the spot's conditions, rank by the user's goals, explain with links to the knowledge base. Rule-based; a language model may only phrase the explanations. |
| **Calendar and journal** | Seasonal tasks for the user's own plants, drawn from the knowledge base, plus a garden journal. |

## Planned order

1. Knowledge base for fruit trees: course fundamentals, then species, varieties, rootstocks, pests and diseases
2. Knowledge base for further plant groups
3. Plot editor and sun map
4. Recommender and planting planner
5. Calendar and journal

## First version scope

- **Content:** fruit trees only. Mistakes with trees are the costliest — a vegetable bed can be restarted every year, a tree cannot.
- **Features:** the knowledge base with optional sign-in for course progress and change notifications. No "my garden" inventory and no user feedback yet.
- **Language:** Hungarian. Other languages may follow later, possibly with AI-assisted translation.
- **Region:** Hungary — its climate, seasons and commonly grown plants.
- **Platform:** web. Mobile apps may come later.

## Open questions

- How mistakes are discovered without user feedback.
- Expert review: who reviews the content and how they are involved.
- Sources beyond textbooks, and how they are verified.
