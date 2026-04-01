# cloudgeo-skills

Repository of Agent Skills for different Cloud Geography tools used with Google Cloud.

## Available Skills

| Skill | Description |
| --- | --- |
| [earthengine](skills/earthengine/SKILL.md) | Expert guidance for Google Earth Engine (GEE) CLI. |
| [bqgeo](skills/bqgeo/SKILL.md) | Expert guidance for BigQuery geospatial analytic queries and best practices. |

## How to Use

Each skill follows the [Agent Skills standard](https://agentskills.io). You can activate a skill manually using the `activate_skill` tool or let your agent automatically identify the need for one based on the skill's description.

### Manual Activation

```bash
/activate_skill earthengine
```

## Contributing

Please follow the Software Development Lifecycle (SDLC) as defined in [GEMINI.md](GEMINI.md). All new skills should be placed in the `skills/` directory and include a `SKILL.md` with appropriate YAML frontmatter.
