# eHotels – Hotel Booking Management System

**HotelHub** is a full-stack hotel booking management system built with modern web technologies. It consists of three main components:

- **PostgreSQL** – Relational database management system (RDBMS)
- **Django** – Python-based backend framework for handling business logic and API requests
- **React** – Frontend JavaScript library for building dynamic user interfaces

---

## 📦 Technologies Used

- **Backend**: Django (Python)
- **Frontend**: React
- **Database**: PostgreSQL
- **Containerization**: Docker & Docker Compose

---

## 🚀 Installation Guide

Follow the steps below to get the application running locally:

### 1. Clone the repository

```bash
git clone https://github.com/malsadev/HotelHub.git

## Running the front end
Run `cd $PROJECT_ROOT/frontend && npm run dev`.
```
### 3. Launch the Backend
```bash
cd csi2532-project/ehotels
docker compose up

```
## SQL schemas

### country
|Attribute  |Type                         |Constraints
|-----------|-----------------------------|-----------------------------
|id         |SERIAL                       |PRIMARY KEY          
|name       |text                         |

### adress
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|id                            |SERIAL                              |PRIMARY KEY   
|street                        |text                                |
|city                          |text                                |
|country_id                    |integer                             |FOREIGN KEY REFERENCES Country(id)
|postalcode                    |text                                |

### hotel_chain
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|id                            |SERIAL                              |PRIMARY KEY   
|name                          |text                                |

### hotel_chain_central_offices
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|hotel_chain_id                |SERIAL                              |FOREIGN KEY REFERENCES hotel_chain(id) ON DELETE CASCADE   
|adress_id                     |integer                             |FOREIGN KEY REFERENCES adress(id) ON DELETE SET NULL

### hotel_chain_contact_emails
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|hotel_chain_id                |SERIAL                              |FOREIGN KEY REFERENCES hotel_chain(id) ON DELETE CASCADE   
|email                         |text                                |

### hotel_chain_contact_numbers
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|hotel_chain_id                |SERIAL                              |FOREIGN KEY REFERENCES hotel_chain(id) ON DELETE CASCADE   
|phone                         |text                                |

### hotel
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|id                            |SERIAL                              |PRIMARY KEY
|hotel_chain_id                |SERIAL                              |FOREIGN KEY REFERENCES hotel_chain(id) ON DELETE CASCADE   
|adress_id                     |integer                             |FOREIGN KEY REFERENES adress(id) ON DELETE SET NULL
|rating                        |integer                             |

### hotel_contact_emails
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|hotel_id                      |SERIAL                              |FOREIGN KEY REFERENCES hotel_chain(id) ON DELETE CASCADE   
|email                         |text                                |

### hotel_contact_numbers
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|hotel_id                      |SERIAL                              |FOREIGN KEY REFERENCES hotel_chain(id) ON DELETE CASCADE   
|phone                         |text                                |

### employee
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|id                            |SERIAL                              |PRIMARY KEY   
|ssn                           |text                                |
|fullname                      |text                                |
|adress_id                     |integer                             |FOREIGN KEY REFERENCES adress(id) ON DELETE SET NULL
|works_in                      |SERIAL                              |FOREIGN KEY REFERENCES hotel(id) ON DELETE CASCADE


### employee_roles
```sql
CREATE TYPE employee_role AS ENUM ('manager', 'director', 'receptionist');
```
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|employee_id                   |SERIAL                              |FOREIGN KEY REFERENCES employee(id) ON DELETE CASCADE
|role                          |employee_role                       |

### room
```sql
CREATE TYPE room_capacity AS ENUM ('simple', 'double');
CREATE TYPE room_view AS ENUM ('ocean', 'mountains');
CREATE TYPE expansion_type AS ENUM ('none', 'additional bed', 'private balcony');
```
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|id                            |SERIAL                              |PRIMARY KEY
|hotel_id                      |SERIAL                              |FOREIGN KEY REFERENCES hotel(id) ON DELETE CASCADE
|price_per_day                 |integer                             |
|surface_area                  |integer                             |
|capacity                      |room_capacity                       |
|damage_description            |text                                |
|expansion                     |expansion_type                      |
|view_type                     |room_view                           |

### room_amenities
```sql
CREATE TYPE amenity_type AS ENUM ('TV', 'Fridge', 'AC');
```
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|room_id                       |SERIAL                              |FOREIGN KEY REFERENCES room(id) ON DELETE CASCADE
|amenity                       |amenity_type                        |
|                              |                                    |PRIMARY KEY (room_id, amenity)

### client
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|id                            |SERIAL                              |PRIMARY KEY
|fullname                      |text                                |
|adress_id                     |integer                             |FOREIGN KEY REFERENCES adress(id) ON DELETE SET NULL
|ssn                           |text                                |
|registration_date             |DATE                                |

### reservation
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|id                            |SERIAL                              |PRIMARY KEY
|client_id                     |integer                             |FOREIGN KEY REFERENCES client(id) ON DELETE SET NULL
|room_id                       |integer                             |FOREIGN KEY REFERENCES room(id) ON DELETE SET NULL
|start_date                    |DATE                                |
|end_date                      |DATE                                |

### rental
|Attribute                     |Type                                |Constraints    
|------------------------------|------------------------------------|------------------------
|id                            |SERIAL                              |PRIMARY KEY
|reservation_id                |integer                             |FOREIGN KEY REFERENCES reservation(id) ON DELETE SET NULL
|payment                       |integer                             |

## INDEXES
```sql
CREATE INDEX postalcode_ndx ON adress (postalcode);
CREATE UNIQUE INDEX employee_ssn_idx ON employee (ssn);
CREATE UNIQUE INDEX client_ssn_idx ON client (ssn);
```

## Triggers
```sql
CREATE OR REPLACE TRIGGER check_conflicting_reservation
    BEFORE INSERT OR UPDATE ON reservation
    FOR EACH ROW
        EXECUTE FUNCTION validate_new_reservation();
CREATE OR REPLACE FUNCTION validate_new_reservation() RETURNS TRIGGER AS
$$
DECLARE
    conflicting_reservations integer;
    conflicting_start_date DATE;
    conflicting_end_date DATE;
BEGIN
    SELECT COUNT(*) FROM reservation WHERE
    reservation.room_id = NEW.room_id AND
    (
        (NEW.start_date >= reservation.start_date AND NEW.start_date < reservation.end_date)
    OR 
        (NEW.end_date > reservation.start_date AND NEW.end_date <= reservation.end_date)
    )
    INTO conflicting_reservations;

    [...]

    IF conflicting_reservations > 0 THEN
        RAISE EXCEPTION 'reservation [%, %] conflicts with reservation [%, %]. Room id: %',
            NEW.start_date, NEW.end_date, conflicting_start_date, conflicting_end_date, NEW.room_id;

    END IF;

    RETURN NEW;
END;
$$
LANGUAGE PLPGSQL;


CREATE OR REPLACE TRIGGER ensure_one_manager_per_hotel
    AFTER INSERT OR UPDATE ON employee_roles
    FOR EACH ROW 
        EXECUTE FUNCTION ensure_unique_manager();
CREATE OR REPLACE FUNCTION ensure_unique_manager() RETURNS TRIGGER AS 
$$
DECLARE
    num_managers integer;
    hotel_id integer;
BEGIN
    SELECT works_in FROM employee
    WHERE employee.id = NEW.employee_id
    INTO hotel_id;

    WITH hotel_employees AS (
        SELECT id, works_in, role 
        FROM employee JOIN employee_roles ON id = employee_id
        WHERE works_in = hotel_id 
    )
    SELECT COUNT(*) FROM hotel_employees
    WHERE role = 'manager'::employee_role
    INTO num_managers;
    


    IF num_managers > 1 THEN
        RAISE EXCEPTION 'Attempting to assign more than one manager to hotel %.', hotel_id;
    END IF;

    RETURN NEW;
END;
$$
LANGUAGE PLPGSQL;

```

 API Documentation

This documentation provides details on how to use the API endpoints for client registration, login, and room search functionalities.

## 1. Register user (client or employee)

- **URL:** `/register`
- **Method:** `POST`
- **Description:** Registers a new client or employee.
- **Request Body Parameters:**
  - `user_type` (string, optional): either "client" or "employee". Default is "client".
  - `fullname` (string, required): Full name of the client.
  - `ssn` (string, required): Social security number of the client.
  - `street` (string، required): Street address of the client.
  - `city` (string, required): City of residence of the client.
  - `country` (string, required): Country of residence of the client.
  - `postal_code` (string, required): Postal code of the client's address.
  - `hotel_id` (integer, optional): main hotel where new employee will be working. Required when `user_type` is "employee".
- **Response:**
  - `201 Created`: Client registered successfully.
  - `400 Bad Request`: Missing required fields.
  - `405 Method Not Allowed`: Only `POST` method allowed.
 
Register Endpoint

    Sample JSON Body:

    json

    {
        "fullname": "John Doe",
        "ssn": "123456789",
        "street": "123 Main Street",
        "city": "Exampleville",
        "country": "Exampleland",
        "postal_code": "12345",
        "user_type": "client",
        "hotel_id": 1 //does not affect behavior when user_type is "client"
    }

    This JSON body registers a new client named "John Doe" with the provided address details in Exampleville, Exampleland. The user is associated with a specific hotel identified by the hotel ID.

## 2. Login User

- **URL:** `/login`
- **Method:** `POST`
- **Description:** Logs in an existing client.
- **Request Body Parameters:**
  - `user_type` (string, optional): either "client" or "employee". Default is "client".
  - `fullname` (string, required): Full name of the client.
  - `ssn` (string, required): Social security number of the client.
- **Response:**
  - `200 OK`: Client login successful. Response includes `user_id`.
  - `401 Unauthorized`: Client login failed.
  - `405 Method Not Allowed`: Only `POST` method allowed.

Login Endpoint

    Sample JSON Body:

    json

    {
        "fullname": "John Doe",
        "ssn": "123456789",
        "user_type": "client"
    }

    This JSON body attempts to log in a client named "John Doe" with the provided SSN. If the credentials are correct, the user is logged in successfully.


## 3. Search Rooms

- **URL:** `/search_rooms`
- **Method:** `GET`
- **Description:** Search available rooms based on specified criteria.
- **Request Body Parameters:**
  - `user_type` (string, optional): User type inititating request. Either "client" or "employee". Default is "client".
  - `user_id` (string, required): ID of the authenticated user.
  - `country_name` (string): Name of the country.
  - `city` (string): City name.
  - `hotel_chain_name` (string): Name of the hotel chain (optional).
  - `hotel_rating` (integer): Minimum hotel rating (optional).
  - `capacity` (integer): Capacity of the room (optional).
  - `room_price` (integer): Maximum price per day of the room (optional).
  - `total_rooms` (integer): Total number of rooms available in the hotel (optional).
  - `res_start_date` (string, format: YYYY-MM-DD): Start date for reservation.
  - `res_end_date` (string, format: YYYY-MM-DD): End date for reservation.
- **Response:**
  - `200 OK`: List of available rooms matching the criteria.
  - `401 Unauthorized`: Client authentication failed.
  - `405 Method Not Allowed`: Only `GET` method allowed.
 
Search Rooms Endpoint

    Sample JSON Body:

    json

    {
        "user_id": 123,
        "user_type": "client",
        "country_name": "Exampleland",
        "city": "Exampleville",
        "hotel_chain_name": "Example Hotel Group",
        "hotel_rating": 4,
        "capacity": 2,
        "room_price": 100,
        "total_rooms": 5,
        "res_start_date": "2024-04-01",
        "res_end_date": "2024-04-05"
    }

    This JSON body represents a room search request by a client in Exampleville, Exampleland, looking for rooms in hotels owned by the "Example Hotel Group" with a rating of 4 or higher, having a capacity of 2 people, priced at $100 per day, with a total of 5 rooms available. The search is conducted for the specified reservation dates.


     Sample Json body with most optional criteria omitted
     
     json

    {
        "user_id": 123,
        "user_type": "client",
        "country_name": "Exampleland",
        "city": "Exampleville",
        "res_start_date": "2024-04-01",
        "res_end_date": "2024-04-05"
    }

 
### 4. Reserve Room
  - **URL:** `/reserve_room`
  - **Method:** `POST`
  - **Description:** `Reserves a room for a specified period.`
  - **Request Body Parameters:**
    - `user_id` (integer): The ID of the user making the reservation.
    - `user_type` (string, optional): The type of user making the reservation (optional). By default "client"
    - `room_id` (integer): The ID of the room to be reserved.
    - `client_id` (integer): The ID of the client making the reservation.
    - `start_date` (string): The start date of the reservation (YYYY-MM-DD format).
    - `end_date` (string): The end date of the reservation (YYYY-MM-DD format).
  - **Response:**
    - `200 OK`: Reservation successful. Response includes the `ID` of the newly created reservation.

Reserve Room Endpoint

    Sample JSON Body:

    json

    {
      "user_id": 123,
      "user_type": "client",
      "room_id": 456,
      "client_id": 789,
      "start_date": "2024-04-01",
      "end_date": "2024-04-05"
    }

This JSON body represents a room reservation request by a client with the specified room ID (456) and client ID (789) for the reservation period from April 1, 2024, to April 5, 2024.


### 5.Rent Room

    URL: /rent_room
    Method: POST
    Description: Rents a room for a specified reservation.
    Request Body Parameters:
        user_id (integer): The ID of the user making the rental.
        user_type (string, optional): The type of user making the rental. Only employees are allowed to rent rooms. Default is "client".
        reservation_id (integer): The ID of the reservation to be rented.
        payment (float): The payment amount for the rental.
    Response:
        200 OK: Rental successful. Response includes the ID of the newly created rental.

     json

    {
    "message": "Rental success",
    "rental": 12345
    }

400 Bad Request: Missing required fields in the request body.

    json

    {
    "message": "Missing required fields"
    }

401 Unauthorized: Client authentication failed.




### 5.Add Room

http post /add_room

sample request body

{
    "user_type": "employee",
    "user_id": 801,
    "hotel_id": 81,
    "price_per_day": 300,
    "surface_area": 300,
    "room_capacity" : "simple",
    "damage_description" : "good",
    "expansion_type" : "none",
    "view_type": "ocean"
}

### 5.Add Hotel

http post /add_hotel

sample request body

{
    "user_type": "employee",
    "user_id": 801,
    "rating": 3,
    "hotel_chain_id": 1,
    "street" : "bab",
    "city" : "heath",
    "country" : "5",
    "postal_code": "postaa"
}
