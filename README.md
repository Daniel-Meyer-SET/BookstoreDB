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
