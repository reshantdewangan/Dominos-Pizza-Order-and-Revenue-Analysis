create table orders (
order_id integer not null,
order_date date not null,
order_time varchar(8) not null,
custid integer not null,
status varchar(9) not null
);


create table customers (
custid integer not null,
first_name varchar(8) not null,
last_name varchar(7) not null,
email varchar(19) not null,
phone bigint not null,
address varchar(11) not null,
city varchar(5) not null,
state varchar(6) not null,
postal_code integer not null
);


create table order_details (
order_details_id integer not null,
order_id integer not null,
pizza_id varchar(14) not null,
quantity integer not null
);


create table pizza_types (
pizza_type_id varchar(50) not null,
name varchar(100) not null,
category varchar(50) not null,
ingredients text not null
);


create table pizzas (
pizza_id varchar(14) not null,
pizza_type_id varchar(14) not null,
size varchar(3) not null,
price numeric(4,2) not null
);





ALTER TABLE pizzas
add constraint pizzas_pkey primary key (pizza_id);

ALTER TABLE orders
add constraint orders_pkey primary key (order_id);

ALTER TABLE customers
add constraint customers_pkey primary key (custid);

ALTER TABLE order_details
add constraint order_details_pkey primary key (order_details_id);

ALTER TABLE pizza_types
add constraint pizza_types_pkey primary key (pizza_type_id);

ALTER TABLE pizzas
add constraint fk_child_parent foreign key (pizza_type_id) references pizza_types(pizza_type_id);

ALTER TABLE orders
add constraint fk_child_parent foreign key (custid) references customers(custid);

ALTER TABLE order_details
add constraint fk_child_parent_product foreign key (pizza_id) references pizzas(pizza_id);

ALTER TABLE order_details
add constraint fk_child_parent foreign key (order_id) references orders(order_id);


