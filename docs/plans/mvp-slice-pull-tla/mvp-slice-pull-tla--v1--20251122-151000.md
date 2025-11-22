# MVP Slice - Pull TLA

## Changelog
- **2025-11-22**: Renamed from `research-and-development-mvp-slice-pull-tla`. Removed prefix.

## Table of Contents

- [Research](#research)
- [Development](#development)
- [Notes](#notes)

## Research

Referring to [https://scryfall.com/docs/api](https://scryfall.com/docs/api), When I call [https://api.scryfall.com](https://api.scryfall.com)

- I must use HTTPS
- I must use the following Request Headers
  - Accept - `Accept: */*`
  - User-Agent - `User-Agent: Scryscraper/0.1`
- I must honour rate limit policies, which is one request ever 50ms, and must cache data for 24 hours. That is, if I have identified that I've already pulled a particular payload and its data, I should not make that request again for 24 hours
- I must honour 429s. e.g. Scryfall, and any other operator of an API can request throttled incoming traffic, and it's within their rights to ban any consumer that does not adhere to fair usage policies

Both scryfall.com and scryfall.io are behind cloudflare. Expect the possibility that cloudflare will preventing access at any time. Cloudflare can and did actually go down during development, anyways, so it's imperative to handle errors appropriately, and to back off when throttled or otherwise unable to get a successful response

- When hashing the request to determine if I have a cached version, I do not want cache hits to include the data from such a throttle. If anything other than a 200 is returned, there is no need to save the response in cache. However, for a response code like 301 and 302, however, follow the redirect. For a response code of 304, do not bother updating the cache, but do set the last requested value to the current time to prevent violation of the 24 hour before requesting again policy. For any other error, including errors in the request, like 4xx or 5xx, also prevent an

Card images are hosted on `.scryfall.io`, which are not advised as rate limited. However, response headers clearly show the usage of Cloudflare, so it's entirely possible that rate limits can be hit there as well. Likewise, Cloudflare can and did actually go down during development, so it's within my own best interest to back off and try again later. . In which case, don't cache the response, but do wait a few minutes before trying again. It might be reasonable to create a jsonc file for every page that has been accessed and cached, and to perhaps hash the url to ensure a deterministic method of hitting cache. The jsonc file can contain a timestamp to say when access was last attempted. Apparently "warnings" can appear in the response, so those should be respected as actual responses and should be cached

- `Cf-Cache-Status`
- `Cf-Ray`

It looks like I will need to make a mask to show the cards. e.g. [Aerial Modification](https://cards.scryfall.io/large/front/b/8/b89cab47-25fb-49ea-bb43-90a0089b6b20.jpg?1576381258) has white in the corners

- Given the response's Content-Type header being `image/jpeg`, I'm not expecting transparency

Here is an example response for a Card (Extracted from a call to Set)

- Full response not listed, but this is the shape. [https://scryfall.com/docs/api/lists](https://scryfall.com/docs/api/lists) shows a definitive shape however. I've added them in as comments

```jsonc
{
  /**
    Type: String
    Comment: Will never be null or missing. Will always be "list"
    Field: "object"
  */
  "object": "list",
  /**
    Type: Integer
    Comment: Can be null or missing
    Field: "total_cards"
  */
  "total_cards": 197,
  /**
    Type: Boolean
    Comment: Will never be null or missing
    Field: "has_more"
  */
  "has_more": true,
  /**
    Type: String (URI)
    Comment: Can be null or missing
    Field: "next_page"
  */
  "next_page": "https://api.scryfall.com/cards/search?format=json&include_extras=true&include_multilingual=false&include_variations=true&order=set&page=2&q=e%3Aaer&unique=prints",
  /**
    Type: Array
    Comment: Will never be null or missing
    Field: "data"
  */
  "data": [] // Returned 5 Cards. See below
  /**
    Type: Array
    Comment: Can be null or missing. Was missing in this case
    Field: "warnings"
  */
}
```

- Each entry in `image_uris` has a query param of `?1576381258`. This might be a timestamp of sorts to when the record was last updated. That is inconclusive however as `1576381258` resolves to `December 15, 2019`, which differs to the `released_at` value of `2017-01-20`
- `include_extras` was set to `true`
- `include_multilingual` was set to `false`. I might like to set that to true in the future
- `include_variations` was set to `true`
- `order` was set to `set`. I'm not 100% what that means
- `page` was set to `2`
- `unique` was set to `prints`, but I'm not sure what that means
- I didn't see a "per page". I wonder if that option exists
- I expect `include_extras`, `include_multilingual`, and `include_variations` will require polymorphism. The base shape will be what comes back from the API with those `include_` flags set to false

```jsonc
{
  "object": "card",
  "id": "b89cab47-25fb-49ea-bb43-90a0089b6b20",
  "oracle_id": "290cd0b1-55d6-4b30-962d-075a6f75d562",
  "multiverse_ids": [423668],
  "mtgo_id": 62601,
  "mtgo_foil_id": 62602,
  "tcgplayer_id": 126445,
  "cardmarket_id": 294845,
  "name": "Aerial Modification",
  "lang": "en",
  "released_at": "2017-01-20",
  "uri": "https://api.scryfall.com/cards/b89cab47-25fb-49ea-bb43-90a0089b6b20",
  "scryfall_uri": "https://scryfall.com/card/aer/1/aerial-modification?utm_source=api",
  "layout": "normal",
  "highres_image": true,
  "image_status": "highres_scan",
  "image_uris": {
    "small": "https://cards.scryfall.io/small/front/b/8/b89cab47-25fb-49ea-bb43-90a0089b6b20.jpg?1576381258",
    "normal": "https://cards.scryfall.io/normal/front/b/8/b89cab47-25fb-49ea-bb43-90a0089b6b20.jpg?1576381258",
    "large": "https://cards.scryfall.io/large/front/b/8/b89cab47-25fb-49ea-bb43-90a0089b6b20.jpg?1576381258",
    "png": "https://cards.scryfall.io/png/front/b/8/b89cab47-25fb-49ea-bb43-90a0089b6b20.png?1576381258",
    "art_crop": "https://cards.scryfall.io/art_crop/front/b/8/b89cab47-25fb-49ea-bb43-90a0089b6b20.jpg?1576381258",
    "border_crop": "https://cards.scryfall.io/border_crop/front/b/8/b89cab47-25fb-49ea-bb43-90a0089b6b20.jpg?1576381258"
  },
  "mana_cost": "{4}{W}",
  "cmc": 5,
  "type_line": "Enchantment — Aura",
  "oracle_text": "Enchant creature or Vehicle\nAs long as enchanted permanent is a Vehicle, it's a creature in addition to its other types.\nEnchanted creature gets +2/+2 and has flying.",
  "colours": ["W"],
  "colour_identity": ["W"],
  "keywords": ["Enchant"],
  "legalities": {
    "standard": "not_legal",
    "future": "not_legal",
    "historic": "not_legal",
    "timeless": "not_legal",
    "gladiator": "not_legal",
    "pioneer": "legal",
    "modern": "legal",
    "legacy": "legal",
    "pauper": "not_legal",
    "vintage": "legal",
    "penny": "not_legal",
    "commander": "legal",
    "oathbreaker": "legal",
    "standardbrawl": "not_legal",
    "brawl": "not_legal",
    "alchemy": "not_legal",
    "paupercommander": "not_legal",
    "duel": "legal",
    "oldschool": "not_legal",
    "premodern": "not_legal",
    "predh": "not_legal"
  },
  "games": ["paper", "mtgo"],
  "reserved": false,
  "game_changer": false,
  "foil": true,
  "nonfoil": true,
  "finishes": ["nonfoil", "foil"],
  "oversized": false,
  "promo": false,
  "reprint": false,
  "variation": false,
  "set_id": "a4a0db50-8826-4e73-833c-3fd934375f96",
  "set": "aer",
  "set_name": "Aether Revolt",
  "set_type": "expansion",
  "set_uri": "https://api.scryfall.com/sets/a4a0db50-8826-4e73-833c-3fd934375f96",
  "set_search_uri": "https://api.scryfall.com/cards/search?order=set&q=e%3Aaer&unique=prints",
  "scryfall_set_uri": "https://scryfall.com/sets/aer?utm_source=api",
  "rulings_uri": "https://api.scryfall.com/cards/b89cab47-25fb-49ea-bb43-90a0089b6b20/rulings",
  "prints_search_uri": "https://api.scryfall.com/cards/search?order=released&q=oracleid%3A290cd0b1-55d6-4b30-962d-075a6f75d562&unique=prints",
  "collector_number": "1",
  "digital": false,
  "rarity": "uncommon",
  "card_back_id": "0aeebaf5-8c7d-4636-9e82-8c27447861f7",
  "artist": "Jung Park",
  "artist_ids": ["269392ac-4c06-4650-98e2-d49a5a7f2371"],
  "illustration_id": "4286ba7e-ed54-4c31-93bf-bed61810cf8f",
  "border_colour": "black",
  "frame": "2015",
  "full_art": false,
  "textless": false,
  "booster": true,
  "story_spotlight": false,
  "edhrec_rank": 17839,
  "penny_rank": 11263,
  "prices": {
    "usd": "0.06",
    "usd_foil": "0.33",
    "usd_etched": null,
    "eur": "0.08",
    "eur_foil": "0.15",
    "tix": "0.03"
  },
  "related_uris": {
    "gatherer": "https://gatherer.wizards.com/Pages/Card/Details.aspx?multiverseid=423668&printed=false",
    "tcgplayer_infinite_articles": "https://partner.tcgplayer.com/c/4931599/1830156/21018?subId1=api&trafcat=tcgplayer.com%2Fsearch%2Farticles&u=https%3A%2F%2Fwww.tcgplayer.com%2Fsearch%2Farticles%3FproductLineName%3Dmagic%26q%3DAerial%2BModification",
    "tcgplayer_infinite_decks": "https://partner.tcgplayer.com/c/4931599/1830156/21018?subId1=api&trafcat=tcgplayer.com%2Fsearch%2Fdecks&u=https%3A%2F%2Fwww.tcgplayer.com%2Fsearch%2Fdecks%3FproductLineName%3Dmagic%26q%3DAerial%2BModification",
    "edhrec": "https://edhrec.com/route/?cc=Aerial+Modification"
  },
  "purchase_uris": {
    "tcgplayer": "https://partner.tcgplayer.com/c/4931599/1830156/21018?subId1=api&u=https%3A%2F%2Fwww.tcgplayer.com%2Fproduct%2F126445%3Fpage%3D1",
    "cardmarket": "https://www.cardmarket.com/en/Magic/Products?idProduct=294845&referrer=scryfall&utm_campaign=card_prices&utm_medium=text&utm_source=scryfall",
    "cardhoarder": "https://www.cardhoarder.com/cards/62601?affiliate_id=scryfall&ref=card-profile&utm_campaign=affiliate&utm_medium=card&utm_source=scryfall"
  }
}
```

## Development

I am making a service to pull all cards from the Scryfall API - Specifically the TLA set for this first vertical slice

This service will be using TypeScript, and the pnpm package manager. Under no circumstances will you deviate from this

Certain actions will take priority over others, and I will make an outbox to queue subsequent actions

- In order of priority
  - Set discoverability - But this is not required for this first vertical slice
  - Card discoverability by set - Prioritised by release date descending. e.g. I want TLA cards discovered before FIN
    - Since there are the `include_extras` and `include_multilingual` query parameters, etc. I will set these _all_ to true. We will use their data later
      - However, I need to verify the size of the payload will all flags set to true. Additionally, I need to understand the
  - Low-res card thumbnails for all cards, by set again. Again, I want TLA thumbnails downloaded before FIN thumbnails
    - I only want the low res image for the base card. e.g. I do not need the low res versions of showcase cards, etc.
  - Low-res card thumbnails for all cards, for all alternative arts, by set again. Again, I want TLA thumbnails downloaded before FIN thumbnails
    - I will potentially make a micro frontend here - which serves a carousel of all available card faces. These can be generated and cached. e.g. published to a CDN, low res thumbnails are base-64 encoded so they appear immediately, with subsequent requests to load in a higher resolution image. At this stage I am ok with briefly seeing lower quality images. Users of the site will at least understand cards are loading in. It's not empty space, and won't cause the page to jump around as the elements of heights update
      - This means each thumbnail will require specifying the max width and max height, even if there is now a higher resolution image in memory. The images will still improve in quality for higher pixel density screens anyway
  - High res images for all cards
    - This is the absolute lowest priority, and I think it's also completely unnecessary, as we'll have a reference to the images on Scryfall's CDN. I believe this isn't violating any policies. There's no point in me re-implementing the CDN side of things here. Scryfall has already done the work

Due to the nature of requests, I will maintain a priority on subsequent requests. e.g. card discoverability is priority one. Meaning, when a request to the card set is made, I want to see absolutely all cards in that set before other actions are performed, e.g. to pull card images. I will eventually utilise a priority queue for this, but for now, I'm not expecting a huge amount of subsequent actions. Additionally, it is not important to ensure disaster recovery. Keeping the actions in memory is fine, and perhaps even truncating subsequent actions to precent running out of memory. Once all cards have been discovered, the next highest priority action is to download the lowest resolution image available to the card, so that it's possible to view the thumbnails of cards while higher resolution images are being fetched in the background

Since I'm not worried about the server crashing, I will have background tasks in place

## Notes

[https://scryfall.com/docs/api/layouts](https://scryfall.com/docs/api/layouts) describes which other properties on a card can be inspected

- To me this sounds like another polymorphic Type to have in TS. I have done this before in my Rethought project

## Planning

Remind me about subtasks

Plans will always be in the `docs/plans` directory, and follow this structure

```md
- docs
  - features
    - overview.md
    - <feature>
      - <feature>--<alignments>
        - <timestamp>--<description>.md
  - plans
    - <plan-name>
      - <task-name>.md
      - pre-requisites
        - <priority-number>--<pre-requisite-task-name>
          - <pre-requisite-task-name>--<timestamp>.md
      - subtasks
        - <priority-number>--<sub-task-name>
          - <sub-task-name>--<timestamp>.md
```

If overview.md is missing, it must be made first and foremost. In line with applying commits one by one to ensure my understanding of the product and its evolution is solid, when joining a new project, I should make this overview.md file from scratch

ALL plans MUST reference a feature alignment. If there is no such feature alignment or one is not referenced, the first pre-requisite will be to create the feature alignment. So for example, in [Development](#development) above, I've noted the priority from the perspective of an engineer. This might be off the mark from a product point of view. So even if I'm making up the feature alignment myself, I need it to constantly challenge myself to think from the product side of things first

Before we begin making plans, make sure that all assumptions are clarified. Any outstanding questions that I've personally noted have answers, and any questions that you have are also answered. Any assumptions are documented. Notes on pre-mortems are specified. e.g. These are potentially risks that can be adopted as technical debt, but I want the choice to address the problem first and update the plan, or work on it in another vertical slice. I will decide what's worth handling now vs later. Always ask me. For example - With caching the response for 24 hours, I do not want to update the card data in my database, and update the card to note the data is potentially stale now. HOWEVER, to begin with, it's not a necessary problem to rectify. This is why we make tests. Additionally, you must make me write all the tests before a plan is executed. For example, with TDD, I can test that card data won't be updated when a 4xx or 5xx response code is received. Additionally, if I receive a 429, I want to make the card as potentially stale if it's already been saved before

As such, I expect the first parts of this vertical slice will be me selecting packages like axios vs using fetch. Selecting packages like Zod. Using fake responses from Scryfall in tests, and NEVER sending a request to Scryfall at all until we're ready for the next steps. Additionally, I will want to define migrations - So I'll need to select an ORM package. The database I intend to use is PostgreSQL, so please ensure there's compatibility there. I've heard of Prisma. There's a new one around as well but I keep forgetting its name. Additionally, I will be using Jest for tests, and Mock Service Workers, etc.

Additionally, I would like to be able to run this locally. To that end, please help me set up a docker container to run my node service in, alongside a PostgreSQL server that doesn't lose the data when the container is stopped. I expect that will be achieved by a volume mount. When I run a test, I expect the PostgreSQL database to be restored from the volume mount. This file is intended to be committed. As such, I only want the SCHEMA present in committed versions of this file. Please create a whitelist and architectural rule so that I can approve each and every row of data that's in the database file. Make the whitelist a jsonc file. Where each subsequent object will be keyed with this naming convention - `table-name--<table-name>`. Each subsequent key in `table-name--<table-name>` will contain the id of the record, and the value will be a string, noting the reasoning for the row being there. Ideally this will be a relatively small file. Base configurations of business logic records is likely all I'll expect in here. I'm happy to see what changes I need to make to this thinking as we progress

Additionally, please point out any cognitive biases that I'm missing. I want to make the most informed decisions, free of recency bias, etc. etc. However, I am not interested in debating the language of this service. It will be TypeScript, and it will use pnpm as the package manager. Additionally, if you're going to request any new packages get added, you will ask me first. I will have to do some independent research to understand the popularity, stability, etc. of the package you're requesting

I want to work in vertical slices. If there is subsequent work to be done, treat it as a decision to be made

EVERY time you require input from me, I expect to have a new markdown file created. Each plan will be in its own subfolder under `docs/plans`. NEVER EVER put a plan in a different directory. I expect the new markdown file will follow this naming convention - `docs/plans/<plan-name>/<plan-name>-<timestamp>.md`. Please use the timestamp format of `Ymd-His`.
