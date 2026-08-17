```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple Rent House</title>

    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            font-family: Arial, sans-serif;
            background: #f4f6f8;
            color: #333;
        }

        header {
            background: #2c3e50;
            color: white;
            padding: 25px;
            text-align: center;
        }

        header h1 {
            margin-bottom: 8px;
        }

        .search-box {
            background: white;
            padding: 20px;
            margin: 25px auto;
            max-width: 1000px;
            border-radius: 10px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);

            display: flex;
            gap: 10px;
            flex-wrap: wrap;
        }

        .search-box input,
        .search-box select,
        .search-box button {
            padding: 12px;
            border: 1px solid #ccc;
            border-radius: 6px;
            font-size: 15px;
        }

        .search-box input {
            flex: 2;
            min-width: 200px;
        }

        .search-box select {
            flex: 1;
            min-width: 150px;
        }

        .search-box button {
            background: #3498db;
            color: white;
            border: none;
            cursor: pointer;
        }

        .search-box button:hover {
            background: #2980b9;
        }

        .house-container {
            max-width: 1000px;
            margin: auto;
            padding: 10px 20px 40px;

            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 25px;
        }

        .house-card {
            background: white;
            border-radius: 10px;
            overflow: hidden;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
            transition: transform 0.2s;
        }

        .house-card:hover {
            transform: translateY(-5px);
        }

        .house-card img {
            width: 100%;
            height: 190px;
            object-fit: cover;
        }

        .house-info {
            padding: 18px;
        }

        .house-info h2 {
            margin-bottom: 10px;
            font-size: 21px;
        }

        .location {
            color: #777;
            margin-bottom: 10px;
        }

        .price {
            color: #e74c3c;
            font-size: 20px;
            font-weight: bold;
            margin-bottom: 10px;
        }

        .details {
            margin-bottom: 15px;
            color: #555;
        }

        .details-button {
            width: 100%;
            padding: 11px;
            background: #27ae60;
            color: white;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-size: 15px;
        }

        .details-button:hover {
            background: #219150;
        }

        .no-result {
            text-align: center;
            grid-column: 1 / -1;
            padding: 40px;
            font-size: 20px;
            color: #777;
        }

        footer {
            background: #2c3e50;
            color: white;
            text-align: center;
            padding: 15px;
        }
    </style>
</head>

<body>

    <header>
        <h1>🏠 Simple Rent House</h1>
        <p>Find your perfect rental house</p>
    </header>

    <div class="search-box">

        <input
            type="text"
            id="searchInput"
            placeholder="Search by location..."
        >

        <select id="priceFilter">
            <option value="all">All Prices</option>
            <option value="10000">Under $10,000</option>
            <option value="15000">Under $15,000</option>
            <option value="20000">Under $20,000</option>
        </select>

        <select id="roomFilter">
            <option value="all">All Rooms</option>
            <option value="1">1 Room</option>
            <option value="2">2 Rooms</option>
            <option value="3">3 Rooms</option>
        </select>

        <button onclick="searchHouses()">Search</button>

    </div>

    <main>
        <div class="house-container" id="houseContainer">
        </div>
    </main>

    <footer>
        <p>© 2026 Simple Rent House</p>
    </footer>


    <script>

        const houses = [
            {
                name: "Modern Apartment",
                location: "Taipei",
                price: 12000,
                rooms: 2,
                image: "https://images.unsplash.com/photo-1522708323590-d24dbb6b0267"
            },
            {
                name: "Cozy Studio",
                location: "Taoyuan",
                price: 8500,
                rooms: 1,
                image: "https://images.unsplash.com/photo-1505693416388-ac5ce068fe85"
            },
            {
                name: "Family House",
                location: "Hsinchu",
                price: 18000,
                rooms: 3,
                image: "https://images.unsplash.com/photo-1564013799919-ab600027ffc6"
            },
            {
                name: "City Apartment",
                location: "Taichung",
                price: 15000,
                rooms: 2,
                image: "https://images.unsplash.com/photo-1493809842364-78817add7ffb"
            },
            {
                name: "Small House",
                location: "Kaohsiung",
                price: 9500,
                rooms: 1,
                image: "https://images.unsplash.com/photo-1484154218962-a197022b5858"
            },
            {
                name: "Luxury Apartment",
                location: "Taipei",
                price: 20000,
                rooms: 3,
                image: "https://images.unsplash.com/photo-1600607687920-4e2a09cf159d"
            }
        ];


        function displayHouses(houseList) {

            const container = document.getElementById("houseContainer");

            container.innerHTML = "";

            if (houseList.length === 0) {

                container.innerHTML =
                    '<div class="no-result">No houses found.</div>';

                return;
            }

            houseList.forEach(function(house) {

                const card = document.createElement("div");

                card.className = "house-card";

                card.innerHTML = `
                    <img src="${house.image}" alt="${house.name}">

                    <div class="house-info">

                        <h2>${house.name}</h2>

                        <p class="location">
                            📍 ${house.location}
                        </p>

                        <p class="price">
                            $${house.price.toLocaleString()} / month
                        </p>

                        <p class="details">
                            🛏️ ${house.rooms} room(s)
                        </p>

                        <button
                            class="details-button"
                            onclick="showDetails('${house.name}', '${house.location}', ${house.price}, ${house.rooms})">
                            View Details
                        </button>

                    </div>
                `;

                container.appendChild(card);
            });
        }


        function searchHouses() {

            const searchText =
                document.getElementById("searchInput")
                .value
                .toLowerCase();

            const price =
                document.getElementById("priceFilter").value;

            const rooms =
                document.getElementById("roomFilter").value;


            const results = houses.filter(function(house) {

                const locationMatch =
                    house.location
                    .toLowerCase()
                    .includes(searchText);

                const priceMatch =
                    price === "all" ||
                    house.price <= Number(price);

                const roomMatch =
                    rooms === "all" ||
                    house.rooms === Number(rooms);

                return locationMatch &&
                       priceMatch &&
                       roomMatch;
            });


            displayHouses(results);
        }


        function showDetails(name, location, price, rooms) {

            alert(
                "🏠 " + name +
                "\n\n📍 Location: " + location +
                "\n💰 Rent: $" + price.toLocaleString() + " / month" +
                "\n🛏️ Rooms: " + rooms
            );
        }


        // Display all houses when the page loads
        displayHouses(houses);

    </script>

</body>
</html>
```
