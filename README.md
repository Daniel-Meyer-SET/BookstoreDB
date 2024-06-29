# BookstoreDB
Daniel Meyer
Supreet Kaur
intial commits
-- Table: public.Purchases

-- DROP TABLE IF EXISTS public."Purchases";

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


        [6/29/2024 2:33 PM] Daniel Clg: -- Table: public.Purchases

-- DROP TABLE IF EXISTS public."Purchases";

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

-- Trigger: trigger_update_amount_spent

-- DROP TRIGGER IF EXISTS trigger_update_amount_spent ON public."Purchases";

CREATE OR REPLACE TRIGGER trigger_update_amount_spent
    AFTER INSERT
    ON public."Purchases"
    FOR EACH ROW
    EXECUTE FUNCTION public.update_amount_spent();

-- Trigger: trigger_update_most_popular_genre

-- DROP TRIGGER IF EXISTS trigger_update_most_popular_genre ON public."Purchases";

CREATE OR REPLACE TRIGGER trigger_update_most_popular_genre
    AFTER INSERT OR DELETE OR UPDATE 
    ON public."Purchases"
    FOR EACH STATEMENT
    EXECUTE FUNCTION public.update_most_popular_genre_trigger();
[6/29/2024 2:38 PM] Daniel Clg: -- Table: public.Publishers

-- DROP TABLE IF EXISTS public."Publishers";

CREATE TABLE IF NOT EXISTS public."Publishers"
(
    "PublisherID" integer NOT NULL GENERATED ALWAYS AS IDENTITY ( INCREMENT 1 START 1 MINVALUE 1 MAXVALUE 2147483647 CACHE 1 ),
    "Name" text COLLATE pg_catalog."default",
    CONSTRAINT "Publishers_pkey" PRIMARY KEY ("PublisherID")
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS public."Publishers"
    OWNER to postgres;
