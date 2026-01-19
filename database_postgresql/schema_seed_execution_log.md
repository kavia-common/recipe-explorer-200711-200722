# Recipe Explorer DB Schema + Seed Execution Log

This log lists the **single-statement** SQL commands executed via `psql -c` to create schema and seed minimal demo data.

Connection used (from `db_connection.txt`):
- `psql postgresql://appuser:dbuser123@localhost:5000/myapp`

Note:
- An attempted connection to port `5001` failed with an invalid SSL negotiation response, so all work proceeded on port `5000`.

## 1) Schema (DDL)

- `CREATE TABLE IF NOT EXISTS categories (id SERIAL PRIMARY KEY, name TEXT UNIQUE NOT NULL);`
- `CREATE TABLE IF NOT EXISTS recipes (id SERIAL PRIMARY KEY, title TEXT NOT NULL, description TEXT, instructions TEXT, category_id INT REFERENCES categories(id));`
- `CREATE TABLE IF NOT EXISTS ingredients (id SERIAL PRIMARY KEY, name TEXT UNIQUE NOT NULL);`
- `CREATE TABLE IF NOT EXISTS recipe_ingredients (recipe_id INT REFERENCES recipes(id), ingredient_id INT REFERENCES ingredients(id), quantity TEXT, PRIMARY KEY (recipe_id, ingredient_id));`
- `CREATE TABLE IF NOT EXISTS ratings (id SERIAL PRIMARY KEY, recipe_id INT REFERENCES recipes(id), user_name TEXT NOT NULL, score INT CHECK (score BETWEEN 1 AND 5), comment TEXT, created_at TIMESTAMPTZ DEFAULT now());`

## 2) Indexes

- `CREATE INDEX IF NOT EXISTS idx_recipes_title ON recipes(title);`
- `CREATE INDEX IF NOT EXISTS idx_ingredients_name ON ingredients(name);`
- `CREATE INDEX IF NOT EXISTS idx_ratings_recipe_id ON ratings(recipe_id);`

## 3) Seed Data (DML)

### categories
- `INSERT INTO categories (name) VALUES ('Breakfast') ON CONFLICT (name) DO NOTHING;`
- `INSERT INTO categories (name) VALUES ('Lunch') ON CONFLICT (name) DO NOTHING;`
- `INSERT INTO categories (name) VALUES ('Dinner') ON CONFLICT (name) DO NOTHING;`

### ingredients
- `INSERT INTO ingredients (name) VALUES ('Eggs') ON CONFLICT (name) DO NOTHING;`
- `INSERT INTO ingredients (name) VALUES ('Milk') ON CONFLICT (name) DO NOTHING;`
- `INSERT INTO ingredients (name) VALUES ('Flour') ON CONFLICT (name) DO NOTHING;`
- `INSERT INTO ingredients (name) VALUES ('Chicken') ON CONFLICT (name) DO NOTHING;`
- `INSERT INTO ingredients (name) VALUES ('Tomato') ON CONFLICT (name) DO NOTHING;`
- `INSERT INTO ingredients (name) VALUES ('Basil') ON CONFLICT (name) DO NOTHING;`

### recipes
- `INSERT INTO recipes (title, description, instructions, category_id) VALUES ('Pancakes', 'Fluffy breakfast pancakes', 'Mix flour, milk, and eggs. Cook on a skillet until golden.', (SELECT id FROM categories WHERE name='Breakfast')) ON CONFLICT DO NOTHING;`
- `INSERT INTO recipes (title, description, instructions, category_id) VALUES ('Grilled Chicken', 'Simple grilled chicken breast', 'Season chicken, grill 6-7 minutes per side until cooked through.', (SELECT id FROM categories WHERE name='Dinner')) ON CONFLICT DO NOTHING;`
- `INSERT INTO recipes (title, description, instructions, category_id) VALUES ('Tomato Basil Pasta', 'Quick pasta with tomato and basil', 'Cook pasta. Warm tomatoes, toss with pasta, add basil.', (SELECT id FROM categories WHERE name='Dinner')) ON CONFLICT DO NOTHING;`

### recipe_ingredients
- `INSERT INTO recipe_ingredients (recipe_id, ingredient_id, quantity) VALUES ((SELECT id FROM recipes WHERE title='Pancakes'), (SELECT id FROM ingredients WHERE name='Flour'), '1 cup') ON CONFLICT DO NOTHING;`
- `INSERT INTO recipe_ingredients (recipe_id, ingredient_id, quantity) VALUES ((SELECT id FROM recipes WHERE title='Pancakes'), (SELECT id FROM ingredients WHERE name='Milk'), '1 cup') ON CONFLICT DO NOTHING;`
- `INSERT INTO recipe_ingredients (recipe_id, ingredient_id, quantity) VALUES ((SELECT id FROM recipes WHERE title='Pancakes'), (SELECT id FROM ingredients WHERE name='Eggs'), '2') ON CONFLICT DO NOTHING;`
- `INSERT INTO recipe_ingredients (recipe_id, ingredient_id, quantity) VALUES ((SELECT id FROM recipes WHERE title='Grilled Chicken'), (SELECT id FROM ingredients WHERE name='Chicken'), '1 breast') ON CONFLICT DO NOTHING;`
- `INSERT INTO recipe_ingredients (recipe_id, ingredient_id, quantity) VALUES ((SELECT id FROM recipes WHERE title='Tomato Basil Pasta'), (SELECT id FROM ingredients WHERE name='Tomato'), '2 cups') ON CONFLICT DO NOTHING;`
- `INSERT INTO recipe_ingredients (recipe_id, ingredient_id, quantity) VALUES ((SELECT id FROM recipes WHERE title='Tomato Basil Pasta'), (SELECT id FROM ingredients WHERE name='Basil'), '1/4 cup') ON CONFLICT DO NOTHING;`

### ratings
- `INSERT INTO ratings (recipe_id, user_name, score, comment) VALUES ((SELECT id FROM recipes WHERE title='Pancakes'), 'DemoUser', 5, 'Perfect weekend breakfast!');`
- `INSERT INTO ratings (recipe_id, user_name, score, comment) VALUES ((SELECT id FROM recipes WHERE title='Grilled Chicken'), 'DemoUser', 4, 'Simple and tasty.');`
- `INSERT INTO ratings (recipe_id, user_name, score, comment) VALUES ((SELECT id FROM recipes WHERE title='Tomato Basil Pasta'), 'DemoUser', 5, 'Fresh flavors and quick to make.');`

## 4) Verification Queries

- `SELECT COUNT(*) AS categories_count FROM categories;`
- `SELECT COUNT(*) AS ingredients_count FROM ingredients;`
- `SELECT COUNT(*) AS recipes_count FROM recipes;`
- `SELECT COUNT(*) AS recipe_ingredients_count FROM recipe_ingredients;`
- `SELECT COUNT(*) AS ratings_count FROM ratings;`
