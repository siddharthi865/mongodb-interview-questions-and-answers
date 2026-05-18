# Set 33

| S.No. | Question                                                                                                                                       |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [Design Amazon-like product catalog with variants and attributes](#question-1-design-amazon-like-product-catalog-with-variants-and-attributes) |

## Question 1. Design Amazon-like product catalog with variants and attributes

This is a classic **MongoDB schema design + scalability interview question**. Let’s design an **Amazon-like Product Catalog** supporting **products, variants, and dynamic attributes**.

### 1. Core Concept (Interview Definition)

An Amazon-like product catalog in MongoDB is typically designed using a **flexible, document-based schema** that supports:

- Products (base item)
- Variants (size, color, configuration, etc.)
- Dynamic attributes (brand, material, specs, etc.)
- High read performance for browsing/searching
- Scalability for millions of SKUs

MongoDB is ideal because it supports **nested documents, arrays, and schema flexibility**, which fits product catalogs perfectly.

### 2. Recommended Schema Design (Hybrid Model)

We use a **hybrid approach**:

- Embed variants inside product (for fast reads)
- Normalize categories if needed for scale

#### Products Collection

```javascript
db.products.insertOne({
  _id: ObjectId(),
  name: "Nike Running Shoes",
  category: "Footwear",
  brand: "Nike",
  basePrice: 5000,
  description: "Lightweight running shoes",

  attributes: {
    material: "Mesh",
    gender: "Unisex",
    sport: "Running",
  },

  variants: [
    {
      sku: "NIKE-RUN-BLK-8",
      color: "Black",
      size: 8,
      price: 5200,
      stock: 25,
      images: ["img1.jpg", "img2.jpg"],
    },
    {
      sku: "NIKE-RUN-WHT-9",
      color: "White",
      size: 9,
      price: 5300,
      stock: 10,
      images: ["img3.jpg"],
    },
  ],

  ratings: {
    avg: 4.5,
    count: 1200,
  },

  createdAt: ISODate(),
  updatedAt: ISODate(),
});
```

### 3. Line-by-Line Explanation

#### Product-level fields

- `_id`: Unique identifier (ObjectId)
- `name`, `brand`, `category`: Used for filtering & search
- `basePrice`: Default price (variants may override)
- `description`: Product detail page content

#### attributes (Flexible schema)

```javascript
attributes: {
  material: "Mesh",
  gender: "Unisex"
}
```

- Schema-less key-value store
- Supports **dynamic filtering (facets)** in e-commerce

Advantage: No need to alter schema for new attributes like "battery life", "RAM", etc.

#### variants array (Critical Design Choice)

Each variant represents a **sellable SKU**:

- `sku`: Unique inventory identifier
- `color`, `size`: Variant dimensions
- `price`: Variant-specific pricing
- `stock`: Inventory tracking
- `images`: Media per variant

Embedded because:

- Faster product page rendering
- Single read fetch
- Variants always accessed with product

#### ratings

- Denormalized for fast display
- Avoids runtime aggregation

### 4. Indexing Strategy (VERY IMPORTANT for interviews)

#### 1. Product search indexes

```javascript
db.products.createIndex({ name: "text", brand: 1, category: 1 });
```

- Enables keyword + structured filtering

#### 2. Variant-level search (if querying variants separately)

```javascript
db.products.createIndex({ "variants.sku": 1 });
db.products.createIndex({ "variants.color": 1 });
db.products.createIndex({ "variants.size": 1 });
```

#### 3. Category browsing

```javascript
db.products.createIndex({ category: 1, brand: 1 });
```

### 5. Real-world Query Examples

#### Fetch product by ID

```javascript
db.products.findOne({ _id: ObjectId("...") });
```

#### Filter by category + attribute

```javascript
db.products.find({
  category: "Footwear",
  "attributes.gender": "Unisex",
});
```

#### Find specific variant (SKU lookup)

```javascript
db.products.findOne({
  "variants.sku": "NIKE-RUN-BLK-8",
});
```

#### Price filtering on variants

```javascript
db.products.find({
  "variants.price": { $lt: 6000 },
});
```

### 6. Performance Optimizations

#### Embedding variants

- Avoids JOIN-like operations
- Ideal for read-heavy workloads

#### When NOT to embed variants

If:

- Variants = thousands per product
- Frequent independent updates
  Then use separate collection:

```bash
products
product_variants (normalized)
```

#### Sharding strategy (for Amazon scale)

Shard key options:

```javascript
{ category: 1, _id: 1 }
```

or

```javascript
{ brand: 1, category: 1 }
```

Goal:

- Even distribution
- Avoid hot shards

### 7. Alternative Normalized Design (Enterprise Scale)

#### products

```javascript
{
  (_id, name, brand, category);
}
```

#### variants

```javascript
{
  (_id, productId, sku, color, size, price, stock);
}
```

Used when:

- Inventory system is highly transactional
- Variants updated independently at scale

### 8. Common Pitfalls

❌ Over-embedding large variant arrays
❌ No indexing on attributes → slow filters
❌ Not denormalizing ratings → expensive aggregation
❌ Ignoring SKU uniqueness constraints

### 9. Follow-up Interview Questions

#### Q1: Why embed variants instead of using a separate collection?

**Answer:** Embedding improves read performance and reduces joins, ideal for product detail pages. But normalization is better when variants scale independently.

#### Q2: How would you support dynamic filtering like Amazon filters?

**Answer:** Use flexible `attributes` field + compound indexes + possibly MongoDB Atlas Search for faceted search.
