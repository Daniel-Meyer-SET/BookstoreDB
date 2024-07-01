 
**Daniel Meyer 8653583**
**Supreet Kaur 8897223**
**Raj Gandhi   8973100**

***ChatGPT was used to assist in the creation of Functions, and Triggers***

**Tables**
 **Authors Table**
```
CREATE TABLE IF NOT EXISTS public."Authors"
(
    "AuthorID" integer NOT NULL GENERATED ALWAYS AS IDENTITY ( INCREMENT 1 START 1 MINVALUE 1 MAXVALUE 2147483647 CACHE 1 ),
    "AuthorName" text COLLATE pg_catalog."default",
    "PowerWriter" boolean,
    CONSTRAINT "Authors_pkey" PRIMARY KEY ("AuthorID")
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS public."Authors"
    OWNER to postgres;
```
##Books Table
```
CREATE TABLE IF NOT EXISTS public."Books"
(
    "BookID" integer NOT NULL GENERATED BY DEFAULT AS IDENTITY ( INCREMENT 1 START 1 MINVALUE 1 MAXVALUE 2147483647 CACHE 1 ),
    "Title" text COLLATE pg_catalog."default",
    "DatePublished" date,
    "BookType" text COLLATE pg_catalog."default",
    "AuthorID" integer,
    "PublisherID" integer,
    "WellReviewed" boolean,
    "GenreID" integer NOT NULL,
    "ISBN" text COLLATE pg_catalog."default",
    CONSTRAINT "Books_pkey" PRIMARY KEY ("BookID"),
    CONSTRAINT "AuthorID" FOREIGN KEY ("AuthorID")
        REFERENCES public."Authors" ("AuthorID") MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
        NOT VALID,
    CONSTRAINT "GenreID" FOREIGN KEY ("GenreID")
        REFERENCES public."Genres" ("GenreID") MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
        NOT VALID,
    CONSTRAINT "PublisherID" FOREIGN KEY ("PublisherID")
        REFERENCES public."Publishers" ("PublisherID") MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
        NOT VALID
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS public."Books"
    OWNER to postgres;

-- Trigger: trigger_update_author_power_writer

-- DROP TRIGGER IF EXISTS trigger_update_author_power_writer ON public."Books";

CREATE OR REPLACE TRIGGER trigger_update_author_power_writer
    AFTER INSERT OR DELETE OR UPDATE 
    ON public."Books"
    FOR EACH ROW
    EXECUTE FUNCTION public.update_author_power_writer_trigger();
```
## Customers Table

```
CREATE TABLE IF NOT EXISTS public."Customers"
(
    "CustomerID" integer NOT NULL GENERATED ALWAYS AS IDENTITY ( INCREMENT 1 START 1 MINVALUE 1 MAXVALUE 2147483647 CACHE 1 ),
    "Name" text COLLATE pg_catalog."default",
    "EmailAddress" text COLLATE pg_catalog."default",
    "PhoneNumber" character varying COLLATE pg_catalog."default",
    "LoyalCustomer" boolean,
    CONSTRAINT "Customers_pkey" PRIMARY KEY ("CustomerID")
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS public."Customers"
    OWNER to postgres;
```
## Genres Table

```
CREATE TABLE IF NOT EXISTS public."Genres"
(
    "GenreID" integer NOT NULL GENERATED ALWAYS AS IDENTITY ( INCREMENT 1 START 1 MINVALUE 1 MAXVALUE 2147483647 CACHE 1 ),
    "GenreName" text COLLATE pg_catalog."default",
    "mostPopular" boolean,
    CONSTRAINT "Genres_pkey" PRIMARY KEY ("GenreID")
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS public."Genres"
    OWNER to postgres;
```

## Publishers Table

```
CREATE TABLE IF NOT EXISTS public."Publishers"
(
    "PublisherID" integer NOT NULL GENERATED ALWAYS AS IDENTITY ( INCREMENT 1 START 1 MINVALUE 1 MAXVALUE 2147483647 CACHE 1 ),
    "Name" text COLLATE pg_catalog."default",
    CONSTRAINT "Publishers_pkey" PRIMARY KEY ("PublisherID")
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS public."Publishers"
    OWNER to postgres;
```
## Purchases Table
```
CREATE TABLE IF NOT EXISTS public."Purchases"
(
    "PurchaseID" integer NOT NULL GENERATED ALWAYS AS IDENTITY ( INCREMENT 1 START 1 MINVALUE 1 MAXVALUE 2147483647 CACHE 1 ),
    "CustomerID" integer,
    "BookID" integer,
    "Price" money,
    "PurchaseDate" date,
    CONSTRAINT "Purchases_pkey" PRIMARY KEY ("PurchaseID"),
    CONSTRAINT "BookID" FOREIGN KEY ("BookID")
        REFERENCES public."Books" ("BookID") MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
        NOT VALID,
    CONSTRAINT "CustomerID" FOREIGN KEY ("CustomerID")
        REFERENCES public."Customers" ("CustomerID") MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
        NOT VALID
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS public."Purchases"
    OWNER to postgres;

-- Trigger: trigger_update_customer_loyalty

-- DROP TRIGGER IF EXISTS trigger_update_customer_loyalty ON public."Purchases";

CREATE OR REPLACE TRIGGER trigger_update_customer_loyalty
    AFTER INSERT OR DELETE OR UPDATE 
    ON public."Purchases"
    FOR EACH ROW
    EXECUTE FUNCTION public.update_customer_loyalty_trigger();

-- Trigger: trigger_update_most_popular_genre

-- DROP TRIGGER IF EXISTS trigger_update_most_popular_genre ON public."Purchases";

CREATE OR REPLACE TRIGGER trigger_update_most_popular_genre
    AFTER INSERT OR DELETE OR UPDATE 
    ON public."Purchases"
    FOR EACH STATEMENT
    EXECUTE FUNCTION public.update_most_popular_genre_trigger();

ALTER TABLE public."Purchases"
    DISABLE TRIGGER trigger_update_most_popular_genre;
```

## Reviews Table
```
CREATE TABLE IF NOT EXISTS public."Purchases"
(
    "PurchaseID" integer NOT NULL GENERATED ALWAYS AS IDENTITY ( INCREMENT 1 START 1 MINVALUE 1 MAXVALUE 2147483647 CACHE 1 ),
    "CustomerID" integer,
    "BookID" integer,
    "Price" money,
    "PurchaseDate" date,
    CONSTRAINT "Purchases_pkey" PRIMARY KEY ("PurchaseID"),
    CONSTRAINT "BookID" FOREIGN KEY ("BookID")
        REFERENCES public."Books" ("BookID") MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
        NOT VALID,
    CONSTRAINT "CustomerID" FOREIGN KEY ("CustomerID")
        REFERENCES public."Customers" ("CustomerID") MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
        NOT VALID
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS public."Purchases"
    OWNER to postgres;

-- Trigger: trigger_update_customer_loyalty

-- DROP TRIGGER IF EXISTS trigger_update_customer_loyalty ON public."Purchases";

CREATE OR REPLACE TRIGGER trigger_update_customer_loyalty
    AFTER INSERT OR DELETE OR UPDATE 
    ON public."Purchases"
    FOR EACH ROW
    EXECUTE FUNCTION public.update_customer_loyalty_trigger();

-- Trigger: trigger_update_most_popular_genre

-- DROP TRIGGER IF EXISTS trigger_update_most_popular_genre ON public."Purchases";

CREATE OR REPLACE TRIGGER trigger_update_most_popular_genre
    AFTER INSERT OR DELETE OR UPDATE 
    ON public."Purchases"
    FOR EACH STATEMENT
    EXECUTE FUNCTION public.update_most_popular_genre_trigger();

ALTER TABLE public."Purchases"
    DISABLE TRIGGER trigger_update_most_popular_genre;
```
**Functions**

### check_loyal_customer
```
CREATE OR REPLACE FUNCTION public.check_loyal_customer(
	customer_id integer,
	min_spent money)
    RETURNS boolean
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE PARALLEL UNSAFE
AS $BODY$
DECLARE
    total_spent MONEY;
	isloyalcustomer BOOLEAN;
BEGIN
    SELECT COALESCE(SUM("Price"), '$0.00')
    INTO total_spent
    FROM "Purchases"
    WHERE "CustomerID" = customer_id
      AND "PurchaseDate" >= CURRENT_DATE - INTERVAL '1 year';
	isloyalcustomer:= total_spent::money::numeric > min_spent::money::numeric;
    RETURN isloyalcustomer;
END;
$BODY$;

ALTER FUNCTION public.check_loyal_customer(integer, money)
    OWNER TO postgres;

```
### check_power_writer
```
CREATE OR REPLACE FUNCTION public.check_power_writer(
	author_id integer,
	min_books integer,
	recent_years integer,
	genre_id integer)
    RETURNS boolean
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE PARALLEL UNSAFE
AS $BODY$
DECLARE
    book_count INT;
BEGIN
    SELECT COUNT(*) INTO book_count
     FROM "Books" WHERE "AuthorID" = author_id
      AND "GenreID" = genre_id
      AND EXTRACT (YEAR FROM "DatePublished") >= EXTRACT(YEAR FROM CURRENT_DATE) - recent_years;

    RETURN book_count > min_books;
END;
$BODY$;

ALTER FUNCTION public.check_power_writer(integer, integer, integer, integer)
    OWNER TO postgres;

```
### check_well_reviewed
```
CREATE OR REPLACE FUNCTION public.check_well_reviewed(
	book_id integer)
    RETURNS boolean
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE PARALLEL UNSAFE
AS $BODY$
DECLARE
    book_rating NUMERIC;
    avg_rating NUMERIC;
BEGIN
    -- Get the average rating for the book
    SELECT AVG("Rating")
    INTO book_rating
    FROM "Reviews"
    WHERE "BookID" = book_id;

    -- Get the overall average rating for all books
    SELECT AVG("Rating")
    INTO avg_rating
    FROM "Reviews";

    -- Return true if book rating is better than overall average rating
    RETURN book_rating > avg_rating;
END;
$BODY$;

ALTER FUNCTION public.check_well_reviewed(integer)
    OWNER TO postgres;
```
## update_most_popular_genre
```
CREATE OR REPLACE FUNCTION public.update_most_popular_genre(
	)
    RETURNS void
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE PARALLEL UNSAFE
AS $BODY$
DECLARE
    max_sales NUMERIC;
    popular_genre_id INT;
BEGIN
    -- Temporary table to store genre sales
	
    CREATE TEMP TABLE temp_genre_sales AS
    SELECT g."GenreID", SUM(p."Price") AS "TotalSales"
    FROM "Genres" g
    LEFT JOIN "Books" b USING ("GenreID")
    LEFT JOIN "Purchases" p USING ("BookID")
    GROUP BY g."GenreID";

--    CREATE TEMP TABLE temp_genre_sales AS
--    SELECT g."GenreID", SUM(p."Price") AS "TotalSales"
--    FROM "Genres" g
--    LEFT JOIN "Books" b USING ("GenreID")
--    LEFT JOIN "Purchases" p USING ("BookID")
--    GROUP BY temp_genere_sales."GenreID";

    -- Find the genre with the maximum total sales
    SELECT MAX("TotalSales") AS max_sales, temp_genre_sales."GenreID" AS popular_genre_id
    INTO max_sales, popular_genre_id
    FROM temp_genre_sales;

    -- Update the mostPopular column based on the genre with maximum sales
    UPDATE "Genres"
    SET "mostPopular" = ("GenreID" = popular_genre_id);

    -- Drop the temporary table
    DROP TABLE IF EXISTS temp_genre_sales;
END;
$BODY$;

ALTER FUNCTION public.update_most_popular_genre()
    OWNER TO postgres;
```
**Trigger Functions**
## update_author_power_writer_trigger
```
CREATE OR REPLACE FUNCTION public.update_author_power_writer_trigger()
    RETURNS trigger
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE NOT LEAKPROOF
AS $BODY$
DECLARE
    is_power_writer BOOLEAN;
BEGIN
	
    	-- Evaluate if the author is a power writer
    	SELECT public.check_power_writer(NEW."AuthorID", 5, 5, 1) -- Example values: min_books = 5, recent_years = 5, genre_id = 1
    	INTO is_power_writer;
    	-- Update the is_power_writer flag in Authors table
    	UPDATE "Authors"
    	SET "PowerWriter" = is_power_writer
    	WHERE "AuthorID" = NEW."AuthorID";
		
    RETURN NEW;
END;

$BODY$;

ALTER FUNCTION public.update_author_power_writer_trigger()
    OWNER TO postgres;

```

## update_books_well_reviewed()
```
CREATE OR REPLACE FUNCTION public.update_books_well_reviewed()
    RETURNS trigger
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE NOT LEAKPROOF
AS $BODY$
BEGIN
    -- Update the 'WellReviewed' flag in the 'Books' table based on the result of check_well_reviewed function
    UPDATE "Books"
    SET "WellReviewed" = public.check_well_reviewed(NEW."BookID")
    WHERE "BookID" = NEW."BookID";

    RETURN NEW;
END;
$BODY$;

ALTER FUNCTION public.update_books_well_reviewed()
    OWNER TO postgres;

```
## update_customer_loyalty_trigger
```
CREATE OR REPLACE FUNCTION public.update_customer_loyalty_trigger()
    RETURNS trigger
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE NOT LEAKPROOF
AS $BODY$
DECLARE
    is_loyal BOOLEAN;
BEGIN
	
	    SELECT public.check_loyal_customer(NEW."CustomerID", '$1000.00') -- spending threshold is 1000
	    INTO is_loyal;
	
	    -- Update the flag in Customers table
	    UPDATE "Customers"
	    SET    "LoyalCustomer" = is_loyal
	    WHERE  "CustomerID" = NEW."CustomerID";
		
    RETURN NEW;
END;
$BODY$;

ALTER FUNCTION public.update_customer_loyalty_trigger()
    OWNER TO postgres;

```
## update_most_popular_genre_trigger
```
CREATE OR REPLACE FUNCTION public.update_most_popular_genre_trigger()
    RETURNS trigger
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE NOT LEAKPROOF
AS $BODY$
BEGIN
    PERFORM public.update_most_popular_genre();
    RETURN NULL;  -- Trigger function must return NULL since it's an AFTER trigger
END;
$BODY$;

ALTER FUNCTION public.update_most_popular_genre_trigger()
    OWNER TO postgres;
```
## update_most_recent_reviews
```
CREATE OR REPLACE FUNCTION public.update_most_recent_reviews()
    RETURNS trigger
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE NOT LEAKPROOF
AS $BODY$
BEGIN
    -- Update mostRecent column based on the 10 most recent reviews for each customer

    UPDATE "Reviews" AS r
    SET "MostRecent" = FALSE
    FROM (
        SELECT "ReviewID"
        FROM (
            SELECT "ReviewID", ROW_NUMBER() OVER (PARTITION BY "CustomerID" ORDER BY "ReviewDate" DESC) AS rank
            FROM "Reviews"
        ) AS ranked_reviews
        WHERE rank > 9
    ) AS recent_reviews
    WHERE r."ReviewID" = recent_reviews."ReviewID";

    RETURN NULL;
END;
$BODY$;

ALTER FUNCTION public.update_most_recent_reviews()
    OWNER TO postgres;

```
