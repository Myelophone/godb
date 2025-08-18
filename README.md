# Godb

postgres + pgx

## Env file

Next variables has to be set via .env file in Your project:

- POSTGRES_USER

- POSTGRES_PASSWORD

- POSTGRES_HOST

- POSTGRES_DB

## Init and usage database

```golang

package main

import (
	"fmt"

	"github.com/myelophone/godb"

	"github.com/joho/godotenv"
)

func main() {
	// if necessary - loading env variables
	if err := godotenv.Load(".env"); err != nil {
		fmt.Println("No .env file found")
	}

	// initiate database
	_, err := godb.InitDB() // can also be dbInstance, err := godb.InitDB()
	if err != nil {
		panic(err)
	}
	defer godb.Close()

	/* check database connection and functionality on init */
	size := 1
	query := fmt.Sprintf(
		"CREATE TABLE IF NOT EXISTS temp_table (first VARCHAR(%d) PRIMARY KEY);",
		size,
	)
	dberr := godb.Exec(query)
	if dberr != nil {
		panic(dberr)
	}

	// inserting
	dberr = godb.Exec(
		"INSERT INTO temp_table (first) VALUES ($1), ($2), ($3);",
		"A", "B", "C",
	)
	if dberr != nil {
		panic(dberr)
	}

	// reading
	rows, err := godb.Query("SELECT * FROM temp_table")
	if err != nil {
		panic(err)
	}
	defer rows.Close()

	for rows.Next() {
		var first string
		if err := rows.Scan(&first); err != nil {
			panic(err)
		}
		fmt.Println(first)
	}

	if err := rows.Err(); err != nil {
		panic(err)
	}

	dberr = godb.Exec("DROP TABLE temp_table;")
	if dberr != nil {
		panic(dberr)
	}

	fmt.Print("Script done.")
}


```
