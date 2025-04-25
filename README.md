

# Auto Delivery Bots #
### - A simple visualisation of automatic delivery robots in real world map ###
<hr>

This is a mini project created for Introduction to AI course in college. The application simulates autonomous delivery bots operating in a constrained environment — **New York Central Park** — using web-based visualization and backend simulation logic.

## 📂 Project Structure

```
Auto Delivery Bots/
│
├── server.py - tkinter backend for GUI
├── autobot_backend_packages.py - Backend helper functions
├── botmap.html - Frontend map interface
├── path_coordinates.json - Sample path data
├── pkg.png - Package icon
├── office.png - sender location icon
├── home.png - destination location icon
```

## 🚀 How to Run

### 1. Clone this repo

```bash
git clone https://github.com/Kishore-Nair/Auto-D-bot.git
```


### 2. Start a Simple HTTP Server for the Frontend

In the project root, run:

```bash
python3 -m http.server 8000
```



### 3. Start the Backend Server

In another terminal, navigate to the project folder and run:

```bash
python3 "Auto Delivery Bots/server.py"
```

This starts the backend server which handles bot data and path logic.

> 📍 Note: This is a *demo project* and location data is limited to locations within **New York Central Park**.

## 📚 Insights

### 🧠 AI & Algorithms
- **Graph Theory**: Applied shortest path and Traveling Salesman Problem (TSP) algorithms using NetworkX.
- **Path Optimization**: Learned how to simulate efficient delivery routes for bots.
- **Geospatial Analysis**: Worked with real-world coordinates using Geopy and OSMnx.

### 🧰 Tools & Libraries
- **OSMnx**: Downloaded and visualized real-world map data (NY Central Park).
- **NetworkX**: Built and analyzed road networks and approximated TSP paths.
- **Tkinter**: Built a GUI for interacting with the backend logic.
- **NumPy**: Handled numerical data and array operations.
- **Webbrowser + JSON**: Integrated frontend map view with backend simulation logic.

### 💻 Technical Skills
- Handled **multi-script integration**: backend logic (Python), frontend map (HTML), and JSON for data sharing.
- Used `http.server` to simulate a basic web server for frontend display.
- Built a system that responds to **user input in GUI** and displays results interactively.

### 🧑‍💻 Project Management
- Designed a **modular backend structure** using separate scripts for backend logic and helper functions.
- Simulated a real-world use-case of delivery bots.
- Understood how different components (UI, logic, data, visualization) work together.




