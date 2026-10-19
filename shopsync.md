# ShopSync Specification

ShopSync is a standard for shops and "sellshops" to broadcast data in order to improve consumer price discovery and user experience.

## Disclaimers

Broadcasting any false or incorrect data is against Rule 1.5. Shops should not broadcast data if they are not connected to a currency network or are inoperable for any other reason. The intent of ShopSync was not for shops to automatically adjust their own prices based on other shops' prices, considering the current lack of any technical protections against falsified data.

Shops may optionally abstain from broadcasting information if they are completely out of stock. Shops supporting this standard can optionally display "ShopSync supported" or "ShopSync-compatible" on their monitors.

## Transmission

Shops should broadcast ShopSync data on channel 9773 using modems or Classic Peripheral radio towers. View the [Classic Peripheral Radio Tower Specification](tower.md) or read below for general information on the ShopSync specification.

Shops broadcasting over modem should use a reply channel of `[computer ID] % 65536` for backwards compatibility with previous ShopSync specifications.

### Transmission Frequency

Any timespans in terms of seconds should be governed by `os.clock()` and `os.startTimer()`.

Shops may broadcast:
- 15 to 30 seconds after the shop turtle/computer starts up
- After the shop inventory has been updated, such as after restocking or after a purchase.
- When the items on sale are changed (price, availability, etc.)
Legacy code may broadcast every 30 seconds rather than the situations described above.

## ShopSync Data Format

See [shopsync.lua](shopsync.lua) for inline comments.

```lua
{
	type = "ShopSync",
	version = 1, -- Major version of the specification
	info = {
		name = "6_4's Shop",
		description = "Shop focused on selling common materials and items.", -- Optional
		owner = "6_4", -- Optional
		computerID = 272,
		multiShop = nil, -- Optional
		features = { "klog", "shipify" }, -- Optional
		software = { -- Optional
			name = "swshop",
			version = "3150525"
		},
		location = {
			coordinates = { 138, 75, 248 },
			description = "North of spawn, just outside Immediate Spawn Area.", -- Optional
			dimension = "overworld" -- Optional, but include this if you are including a location.
		}
	},
	items = {
		{
			prices = {
				{
					value = 100,
					currency = "KRO",
					address = "dia@shop.kro",
					requiredMeta = "sussy"
				}
			},
			item = {
				name = "minecraft:diamond",
				nbt = nil,
				displayName = "Diamond",
				description = nil
			},
			features = { "shipify" }, -- Optional
			dynamicPrice = false, -- Optional
			stock = 100, -- Optional
			madeOnDemand = false, -- Optional
			requiresInteraction = false -- Optional
		},
		{ -- "Sellshop" listing
			shopBuysItem = true,
			prices = {
				{
					value = 10,
					currency = "KRO"
				}
			},
			item = {
				name = "minecraft:gold_ingot",
				nbt = nil,
				displayName = "Gold Ingot",
				description = nil
			},
			features = { "shipify:receive" }, -- Optional
			stock = 100,
			noLimit = false -- Specify to true to indicate the shop has no limit on the number of items it can accept (other than the shop's currency balance).
		},
	}
}
```

### ShopSync Packet

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `type` | string | Yes | Must be "ShopSync" |
| `version` | number | Yes* | Must be 1 |
| `info` | [ShopSyncInfo](#shopsyncinfo) | Yes | Shop information |
| `items` | [ShopSyncItem](#shopsyncitem)[] | Yes | Shop items |

\* `version` may be nil, implying specification of v1.1 or earlier. v1.2 or later must include a `version` field of 1.

### ShopSyncInfo

| Field | Type | Required | Description | Example |
| --- | --- | --- | --- | --- |
| `name` | string | Yes | Shop name | `6_4's Shop` |
| `description` | string | No | Brief shop description. Avoid including information that is already available from other fields | `Shop focused on selling common materials and items.` |
| `owner` | string | No | Shop owner | `6_4` |
| `computerID` | number | Yes* | Shop computer ID | `272` |
| `multiShop` | number | No | If a single computer/turtle operates multiple shops (or a shop and a sellshop), permanent unique integer ID assigned to differentiate them. | `1` |
| `features` | string[] | No | Shop features | `["shipify", "klog"]` |
| `software` | [ShopSyncSoftware](#shopsyncsoftware) | No | Shop software | See [ShopSyncSoftware](#shopsyncsoftware) |
| `location` | [ShopSyncLocation](#shopsynclocation) | No | Shop location | See [ShopSyncLocation](#shopsynclocation) |
| `otherLocations` | [ShopSyncLocation](#shopsynclocation)[] | No | If the shop has additional locations, *pulling from the same stock/items*, this table can contain other locations in an identical format to the location table. | See [ShopSyncLocation](#shopsynclocation) |

\* Some legacy shops may not have a `computerID` field, in which case it can be inferred from the modem reply channel.

### ShopSyncSoftware

| Field | Type | Required | Description | Example |
| --- | --- | --- | --- | --- |
| `name` | string | No | Name of the shop software | `swshop` |
| `version` | string | No | Any human-readable representation of the shop software version (compile date, git commit shorthash, version number) | `3150525` |

### ShopSyncLocation

| Field | Type | Required | Description | Example |
| --- | --- | --- | --- | --- |
| `coordinates` | number[] | No | Coordinates of the location as `[x, y, z]` | `[138, 75, 248]` |
| `description` | string | No | Location description | `North of spawn, just outside immediate spawn area` |
| `dimension` | 'overworld', 'nether', 'end' | No | Dimension of the location | `overworld` |

If you are providing a location, it is recommended to include all 3 fields: `coordinates`, `description`, and `dimension`. Coordinates should be near the shop (where items dispense or where monitors are visible). Coordinates can also be automatically determined via modem GPS if not explicitly configured.

Shops can contain multiple listings for the same item with separate stocks and prices. Out-of-stock listings (`stock = 0`) can be broadcast; ideally, shops should broadcast them based on whether they are displayed on the shop monitor.

### ShopSyncItem

| Field | Type | Required | Description | Example |
| --- | --- | --- | --- | --- |
| `prices` | [ShopSyncPrice](#shopsyncprice)[] | No | List of tables that include price/currency/address information | See [ShopSyncPrice](#shopsyncprice) |
| `item` | [ShopSyncItemDetail](#shopsyncitemdetail) | No | Table describing the item | See [ShopSyncItemDetail](#shopsyncitemdetail) |
| `features` | string[] | No | List of features of the item. This generally overwrites shop-level features. | `["shipify:receive"]` |
| `dynamicPrice` | boolean | No | If false or nil, the full stock can be purchased at the specified price. If true, the price is dynamic and only the first item can be guaranteed at this price. | `false` |
| `madeOnDemand` | boolean | No | Set to true if the item is crafted, processed, or otherwise made on demand. | `false` |
| `requiresInteraction` | boolean | No | Set to true if the shop requires interaction to purchase the item (for example, monitor interaction or chatbox). This **does not apply** if `shopBuysItem` is true. | `false` |
| `shopBuysItem` | boolean | No | Set to true if the item is bought by the shop, false or nil if it is sold by the shop. | `false` |
| `noLimit` | boolean | No | Set to true if the shop is willing to accept more items than it can actually pay out for. Only applies if `shopBuysItem` is **true** (usually false/nil when `dynamicPrice` is true). | `false` |
| `stock` | number | Required unless `madeOnDemand` is true | The number of items in stock, or the limit the shop is willing to purchase if `shopBuysItem` is true. If a sellshop has no specific item limit, it can calculate stock as current balance divided by price (rounded down). | `100` |

### ShopSyncItemDetail

| Field | Type | Required | Description | Example |
| --- | --- | --- | --- | --- |
| `name` | string | Yes | The name of the item as given in `list()`. | `"minecraft:diamond"` |
| `displayName` | string | Yes | The display name of the item, ideally matching what is shown in the shop interface. | `"Diamond"` |
| `description` | string | No | Brief description of the item. | `nil` |
| `nbt` | string | No | The NBT hash of the item if present, or `nil`. | `"adbde11655c5fd620ddec5a55e81ba55"` |

### ShopSyncPrice

| Field | Type | Required | Description | Example |
| --- | --- | --- | --- | --- |
| `value` | number | Yes | The price for 1 item. Can be 0 if the item is free (in which case currency can be set to nil / ignored by readers). | `100` |
| `currency` | string | Yes* | The currency of the price (e.g. `"KRO"`, `"KST"`, `"XLP"`). Can be nil if `value` is 0. | `"KRO"` |
| `address` | string | No | Address which shop users pay to. | `"dia@shop.kro"` |
| `requiredMeta` | string | No | Metadata which shop users need to include in the transaction (e.g. for `/pay <address> <amt> <requiredMeta>`). This should not include anything that goes in the `address` field. | `"sussy"` |
