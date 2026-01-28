# Agent Skills

A collection of skills for AI coding agents. Skills are packaged instructions and scripts that extend agent capabilities.

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Available Skills

### booking

Search Booking.com for hotels with comprehensive filtering. Considers city, dates, price, neighborhoods, and preferred chains (including all major brands). Returns direct hotel website links.

**Use when:**
- "Find hotels in [city]"
- "Search for hotels for [dates]"
- "Book a hotel near [location]"
- Planning travel accommodations

**Features:**
- Searches Booking.com with customizable filters
- Supports all major chains (Hilton, Marriott, IHG, Hyatt, Choice, Best Western)
- Finds direct booking links for potential savings
- Family travel integration with preferences support
- Validates availability, ratings, and amenities

**Search criteria:**
- City/Location and neighborhood
- Check-in/Check-out dates
- Number of adults and children
- Price range per night
- Preferred hotel chains
- Required amenities (pool, fridge, free cancellation, etc.)

**Output:**
```
### Hotel Search Results: Portland, Aug 2-4

#### Option 1: Hampton Inn Portland
- **Brand**: Hampton by Hilton
- **Price**: $189 per night ($378 total)
- **Rating**: 8.2/10 (1,247 reviews)
- **Amenities**: Pool, fridge, free breakfast
- **Booking.com Link**: [URL]
- **Direct Website**: [URL]
```

## Installation

```bash
npx add-skill saguiitay/skills
```

## Usage

Skills are automatically available once installed. The agent will use them when relevant tasks are detected.

**Examples:**
```
Deploy my app
```
```
Review this React component for performance issues
```
```
Help me optimize this Next.js page
```

## Skill Structure

Each skill contains:
- `SKILL.md` - Instructions for the agent
- `scripts/` - Helper scripts for automation (optional)
- `references/` - Supporting documentation (optional)

## License

MIT
