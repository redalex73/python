#!/usr/bin/env python
# coding: utf-8

# ## Simulación de Ciudad: Clientes y Vehículos
# ### Clase Agent

# In[21]:


# Diccionario global para almacenar agentes
agents = {}

class Agent:
    """Clase base para todos los agentes en el sistema."""
    def __init__(self, name):
        self.name = name

    def describe(self):
        return f"Agent: {self.name}"


# ### Clase Client

# In[23]:


class Client(Agent):
    """Clase que representa a un cliente que interactúa con el vehiculo."""
    def __init__(self, name):
        super().__init__(name)
        self.owned_vehicles = []  # Vehículos que posee el cliente
        self.current_vehicle = None  # Vehículo en el que está actualmente
        self.inside_vehicle = False  # Estado de si está dentro de un vehículo

    def buy(self, vehicle_name):
        if vehicle_name in agents and isinstance(agents[vehicle_name], Vehicle):
            vehicle = agents[vehicle_name]
            if not vehicle.is_sold:
                vehicle.is_sold = True
                self.owned_vehicles.append(vehicle_name)
                print(f'Client {self.name} bought vehicle {vehicle_name}.')
            else:
                print(f'Vehicle {vehicle_name} is not available for purchase.')
        else:
            print(f'Vehicle {vehicle_name} not found.')

    def sell(self, vehicle_name):
        if vehicle_name in agents and isinstance(agents[vehicle_name], Vehicle):
            vehicle = agents[vehicle_name]
            if vehicle_name in self.owned_vehicles and not vehicle.is_moving and len(vehicle.passengers) == 0:
                if self.current_vehicle == vehicle_name:
                    self.get_out(vehicle_name)
                vehicle.is_sold = False
                self.owned_vehicles.remove(vehicle_name)
                print(f'Client {self.name} sold vehicle {vehicle_name}.')
            else:
                print(f'Vehicle {vehicle_name} is not available for being sold.')
        else:
            print(f'Vehicle {vehicle_name} not found.')

    def get_in(self, vehicle_name):
        if vehicle_name in agents and isinstance(agents[vehicle_name], Vehicle):
            vehicle = agents[vehicle_name]
            if vehicle_name in self.owned_vehicles and not self.inside_vehicle:
                self.current_vehicle = vehicle_name
                self.inside_vehicle = True
                print(f'Client {self.name} got in vehicle {vehicle_name}.')
            else:
                print(f'Vehicle {vehicle_name} is not available for getting in.')
        else:
            print(f'Vehicle {vehicle_name} not found.')

    def get_out(self, vehicle_name):
        if vehicle_name in agents and isinstance(agents[vehicle_name], Vehicle):
            vehicle = agents[vehicle_name]
            if vehicle_name in self.owned_vehicles and self.inside_vehicle and vehicle_name == self.current_vehicle \
            and not vehicle.is_moving:
                self.current_vehicle = None
                self.inside_vehicle = False
                print(f'Client {self.name} got out of vehicle {vehicle_name}.')
            else:
                print(f'Vehicle {vehicle_name} is not available for getting out.')
        else:
            print(f'Vehicle {vehicle_name} not found.')

    def pick_up_passenger(self, vehicle_name, passenger_name):
        if vehicle_name in agents and isinstance(agents[vehicle_name], Vehicle):
            if passenger_name in agents and isinstance(agents[passenger_name], Client):
                vehicle = agents[vehicle_name]
                passenger = agents[passenger_name]
                if vehicle_name in self.owned_vehicles and self.inside_vehicle and not vehicle.is_moving and not passenger.inside_vehicle:
                    vehicle.passengers.append(passenger_name)
                    passenger.current_vehicle = vehicle_name
                    passenger.inside_vehicle = True
                    print(f'Client {self.name} picked up passenger {passenger_name} in vehicle {vehicle_name}.')
                else:
                    print(f'Passenger {passenger_name} cannot get in vehicle {vehicle_name}.')
            else:
                print(f'Passenger {passenger_name} not found.')
        else:
            print(f'Vehicle {vehicle_name} not found.')

    def drop_off_passenger(self, vehicle_name, passenger_name):
        if vehicle_name in agents and isinstance(agents[vehicle_name], Vehicle):
            if passenger_name in agents and isinstance(agents[passenger_name], Client):
                vehicle = agents[vehicle_name]
                passenger = agents[passenger_name]
                if vehicle_name in self.owned_vehicles and self.inside_vehicle and not vehicle.is_moving and passenger.inside_vehicle:
                    vehicle.passengers.remove(passenger_name)
                    passenger.current_vehicle = None
                    passenger.inside_vehicle = False
                    print(f'Client {self.name} dropped off passenger {passenger_name} in vehicle {vehicle_name}.')
                else:
                    print(f'Passenger {passenger_name} cannot get out of vehicle {vehicle_name}.')
            else:
                print(f'Passenger {passenger_name} not found.')
        else:
            print(f'Vehicle {vehicle_name} not found.')

    def move(self, vehicle_name):
        if vehicle_name in agents and isinstance(agents[vehicle_name], Vehicle):
            vehicle = agents[vehicle_name]
            if vehicle_name in self.owned_vehicles and self.inside_vehicle and vehicle_name == self.current_vehicle \
            and not vehicle.is_moving and not vehicle.is_broken and vehicle.fuel > 0:
                vehicle.is_moving = True
                vehicle.fuel -= 5
                print(f'Client {self.name} started moving vehicle {vehicle_name}.')
            else:
                print(f'Vehicle {vehicle_name} is not available for moving.')
        else:
            print(f'Vehicle {vehicle_name} not found.')

    def stop(self, vehicle_name):
        if vehicle_name in agents and isinstance(agents[vehicle_name], Vehicle):
            vehicle = agents[vehicle_name]
            if vehicle_name in self.owned_vehicles and self.inside_vehicle and vehicle_name == self.current_vehicle \
            and vehicle.is_moving:
                vehicle.is_moving = False
                print(f'Client {self.name} stopped moving vehicle {vehicle_name}.')
            else:
                print(f'Vehicle {vehicle_name} is not available for stopping.')
        else:
            print(f'Vehicle {vehicle_name} not found.')

    def show_in_system(self):
        print("Clients in the system:")
        for name in agents:
            if isinstance(agents[name], Client):
                print(f"{name}")

    def show_with_vehicle(self):
        print("Clients with vehicles:")
        for name in agents:
            if isinstance(agents[name], Client) and agents[name].owned_vehicles:
                print(f'- {self.name} owns {self.owned_vehicles}')

    def show_inside_vehicle(self):
        print("Clients inside vehicles:")
        for name in agents:
            if isinstance(agents[name], Client) and agents[name].inside_vehicle:
                print(f'- {agents[name].name} is inside {agents[name].current_vehicle}')


# ### Clase Vehicle

# In[25]:


class Vehicle(Agent):
    """Clase que representa un vehículo."""
    def __init__(self, name):
        super().__init__(name)
        self.is_sold = False  # Estado de si el vehículo ha sido vendido
        self.is_moving = False  # Estado de movimiento
        self.fuel = 100  # Nivel de combustible
        self.is_broken = False  # Estado de si el vehículo está descompuesto
        self.passengers = [] # Lista de pasajeros del vehículo

    def refuel(self):
        if self.fuel < 100:
            self.fuel = 100
            print(f'Vehicle {self.name} refueled.')
        else:
            print(f'Vehicle {self.name} already has full fuel.')

    def stop_traffic_light(self):
        if self.is_sold and self.fuel > 0 and self.is_moving:
            self.is_moving = False
            print(f'Vehicle {self.name} stopped at the traffic light.')
        else:
            print(f'Vehicle {self.name} cannot stop at traffic light.')

    def get_back_on_track(self):
        if self.is_sold and not self.is_moving and not self.is_broken and self.fuel > 0:
            self.is_moving = True
            self.fuel -= 5
            print(f'Vehicle {self.name} resumed movement.')
        else:
            print(f'Vehicle {self.name} cannot resume movement.')

    def breakdown(self):
        self.is_broken = True
        self.is_moving = False
        print(f'Vehicle {self.name} broke down.')


# ### Clase CitySimulation

# In[27]:


class CitySimulation:
    """Clase principal para gestionar la simulación de la ciudad."""
    def add_agent(self, agent_type, agent_name):
        """Añade un nuevo agente al sistema."""
        if agent_type == 'client':
            agents[agent_name] = Client(agent_name)
        elif agent_type == 'vehicle':
            agents[agent_name] = Vehicle(agent_name)
        print(f'{agent_type.capitalize()} {agent_name} added to the system.')

    def list_agents(self):
        """Muestra todos los agentes en el sistema."""
        print("Current agents:")
        for agent in agents.values():
            print(agent.describe())

    def list_vehicles(self):
        """Muestra todos los vehículos en el sistema."""
        print("Vehicles in the system:")
        for name, agent in agents.items():
            if isinstance(agent, Vehicle):
                print(f'- {agent.name} (Sold: {agent.is_sold})')

    def help(self):
        """Muestra la lista de comandos disponibles."""
        print("""
            Available commands:
            - client add <client_name>: Agregar un cliente al sistema.
            - vehicle add <vehicle_name>: Agregar un vehículo al sistema.
            - client buy <client_name> <vehicle_name>: Añadir un vehículo a un cliente.
            - client get_in <client_name> <vehicle_name>: Subir cliente a vehículo.
            - client sell <client_name> <vehicle_name>: Retirar vehículo del cliente.
            - client get_out <client_name> <vehicle_name>: Bajar cliente de vehículo
            - client move <client_name> <vehicle_name>: Comenzar el movimiento.
            - vehicle refuel <vehicle_name>: Reabastecer el vehículo.
            - client pick_up_passenger <client_name> <vehicle_name> <passenger_name>: Subir pasajero a vehículo
            - client drop_off_passenger <client_name> <vehicle_name> <passenger_name>: Bajar pasajero de vehículo.
            - client stop <client_name> <vehicle_name>: Detener el movimiento.
            - vehicle stop_traffic_light <vehicle_name>: Detener el tráfico en un semáforo.
            - vehicle get_back_on_track <vehicle_name>: Retomar el movimiento.
            - vehicle show_in_system: Mostrar vehículos en el sistema.
            - vehicle show_moving: Mostrar vehículos actualmente en movimiento.
            - vehicle show_without_fuel: Mostrar vehículos que están sin combustible.
            - client show_in_system: Mostrar clientes en el sistema.
            - client show_with_vehicle: Mostrar clientes que poseen vehículos.
            - client show_inside_vehicle: Mostrar clientes actualmente dentro de un vehículo.
            - vehicle random_breakdown <vehicle_name>: Marcar aleatoriamente un vehículo como descompuesto.
            - q: Exit the simulation.
            """)

    def command_loop(self):
        """Bucle principal para gestionar comandos del usuario."""
        print("Starting city simulation... Type 'q' to exit")
        while True:
            command = input('> ')
            if command == 'q':
                break
            self.process_command(command)

    def process_command(self, command):
        """Procesa los comandos ingresados por el usuario."""
        parts = command.split()
        if not parts:
            return
    
        cmd = parts[0]
    
        if cmd == '?':
            self.help()
            return

        elif cmd == 'client':
            if len(parts) > 1:
                if parts[1] == 'add':
                    try:
                        _, _, client_name = parts
                        if client_name in agents and isinstance(agents[client_name], Client):
                            print(f"Client {client_name} already exists.")
                        else:
                            self.add_agent('client', client_name)
                    except ValueError:
                        print("Error: Invalid add command format. Use 'client add <client_name>'")

                elif parts[1] == 'buy':
                    try:
                        _, _, client_name, vehicle_name = parts
                        if client_name in agents and isinstance(agents[client_name], Client):
                            agents[client_name].buy(vehicle_name)
                        else:
                            print(f"Client {client_name} not found.")
                    except ValueError:
                        print("Error: Invalid buy command format. Use 'client buy <client_name> <vehicle_name>'")

                elif parts[1] == 'sell':
                    try:
                        _, _, client_name, vehicle_name = parts
                        if client_name in agents and isinstance(agents[client_name], Client):
                            agents[client_name].sell(vehicle_name)
                        else:
                            print(f"Client {client_name} not found.")
                    except ValueError:
                        print("Error: Invalid sell command format. Use 'client sell <client_name> <vehicle_name>'")

                elif parts[1] == 'get_in':
                    try:
                        _, _, client_name, vehicle_name = parts
                        if client_name in agents and isinstance(agents[client_name], Client):
                            agents[client_name].get_in(vehicle_name)
                        else:
                            print(f"Client {client_name} not found.")
                    except ValueError:
                        print("Error: Invalid get_in command format. Use 'client get_in <client_name> <vehicle_name>'")

                elif parts[1] == 'get_out':
                    try:
                        _, _, client_name, vehicle_name = parts
                        if client_name in agents and isinstance(agents[client_name], Client):
                            agents[client_name].get_out(vehicle_name)
                        else:
                            print(f"Client {client_name} not found.")
                    except ValueError:
                        print("Error: Invalid get_out command format. Use 'client get_out <client_name> <vehicle_name>'")

                elif parts[1] == 'pick_up_passenger':
                    try:
                        _, _, client_name, vehicle_name, passenger_name = parts
                        if client_name in agents and isinstance(agents[client_name], Client):
                            agents[client_name].pick_up_passenger(vehicle_name, passenger_name)
                        else:
                            print(f"Client {client_name} not found.")
                    except ValueError:
                        print("Error: Invalid pick_up_passenger command format. Use 'client pick_up_passenger <client_name> <vehicle_name> <passenger_name>'")

                elif parts[1] == 'drop_off_passenger':
                    try:
                        _, _, client_name, vehicle_name, passenger_name = parts
                        if client_name in agents and isinstance(agents[client_name], Client):
                            if passenger_name in agents and isinstance(agents[passenger_name], Client):
                                agents[client_name].drop_off_passenger(vehicle_name, passenger_name)
                            else:
                                print(f"Passenger {passenger_name} not found.")
                        else:
                            print(f"Client {client_name} not found.")
                    except ValueError:
                        print("Error: Invalid drop_off_passenger command format. Use 'client drop_off_passenger <client_name> <vehicle_name> <passenger_name>'")

                elif parts[1] == 'move':
                    try:
                        _, _, client_name, vehicle_name = parts
                        if client_name in agents and isinstance(agents[client_name], Client):
                            agents[client_name].move(vehicle_name)
                        else:
                            print(f"Client {client_name} not found.")
                    except ValueError:
                        print("Error: Invalid move command format. Use 'client move <client_name> <vehicle_name>'")

                elif parts[1] == 'stop':
                    try:
                        _, _, client_name, vehicle_name = parts
                        if client_name in agents and isinstance(agents[client_name], Client):
                            agents[client_name].stop(vehicle_name)
                        else:
                            print(f"Client {client_name} not found.")
                    except ValueError:
                        print("Error: Invalid stop command format. Use 'client stop <client_name> <vehicle_name>'")

                elif parts[1] == 'show_in_system':
                    client_name = next((name for name in agents if isinstance(agents[name], Client)), None)
                    if client_name:
                        agents[client_name].show_in_system()
                    else:
                        print("No clients found.")

                elif parts[1] == 'show_with_vehicle':
                    client_name = next((name for name in agents if isinstance(agents[name], Client)), None)
                    if client_name:
                        agents[client_name].show_with_vehicle()
                    else:
                        print("No clients with vehicles found.")

                elif parts[1] == 'show_inside_vehicle':
                    client_name = next((name for name in agents if isinstance(agents[name], Client)), None)
                    if client_name:
                        agents[client_name].show_inside_vehicle()
                    else:
                        print("No clients inside vehicles found.")
                else:
                    print("Unknown command. Type '?' for a list of commands.")
            else:
                print("Unknown command. Type '?' for a list of commands.")

        elif cmd == 'vehicle':
            if len(parts) > 1:
                if parts[1] == 'add':
                    try:
                        _, _, vehicle_name = parts
                        if vehicle_name in agents and isinstance(agents[vehicle_name], Vehicle):
                            print(f"Vehicle {vehicle_name} already exists.")
                        else:
                            self.add_agent('vehicle', vehicle_name)
                    except ValueError:
                        print("Error: Invalid add command format. Use 'vehicle add <vehicle_name>'")

                elif parts[1] == 'refuel':
                    try:
                        _, _, vehicle_name = parts
                        if vehicle_name in agents and isinstance(agents[vehicle_name], Vehicle):
                            agents[vehicle_name].refuel()
                        else:
                            print(f'Vehicle {vehicle_name} not found.')
                    except ValueError:
                        print("Error: Invalid refuel command format. Use 'vehicle refuel <vehicle_name>'")        

                elif parts[1] == 'stop_traffic_light':
                    try:
                        _, _, vehicle_name = parts
                        if vehicle_name in agents and isinstance(agents[vehicle_name], Vehicle):
                            agents[vehicle_name].stop_traffic_light()
                        else:
                            print(f'Vehicle {vehicle_name} not found.')
                    except ValueError:
                        print("Error: Invalid stop_traffic_light command format. Use 'vehicle stop_traffic_light <vehicle_name>'") 

                elif parts[1] == 'get_back_on_track':
                    try:
                        _, _, vehicle_name = parts
                        if vehicle_name in agents and isinstance(agents[vehicle_name], Vehicle):
                            agents[vehicle_name].get_back_on_track()
                        else:
                            print(f'Vehicle {vehicle_name} not found.')
                    except ValueError:
                        print("Error: Invalid get_back_on_track command format. Use 'vehicle get_back_on_track <vehicle_name>'") 

                elif parts[1] == 'show_in_system':
                    self.list_vehicles()

                elif parts[1] == 'show_moving':
                    print("Vehicles currently moving:")
                    for name, agent in agents.items():
                        if isinstance(agent, Vehicle) and agent.is_moving:
                            print(f'- {name}')

                elif parts[1] == 'show_without_fuel':
                    print("Vehicles without fuel:")
                    for name, agent in agents.items():
                        if isinstance(agent, Vehicle) and agent.fuel == 0:
                            print(f'- {name}')

                elif parts[1] == 'random_breakdown':
                    try:
                        _, _, vehicle_name = parts
                        if vehicle_name in agents and isinstance(agents[vehicle_name], Vehicle):
                            agents[vehicle_name].breakdown()
                        else:
                            print(f'Vehicle {vehicle_name} not found.')
                    except ValueError:
                        print("Error: Invalid random_breakdown command format. Use 'vehicle random_breakdown <vehicle_name>'") 
                else:
                    print("Unknown command. Type '?' for a list of commands.")
            else:
                print("Unknown command. Type '?' for a list of commands.")
        else:
            print("Unknown command. Type '?' for a list of commands.")


# ### Main

# In[ ]:


if __name__ == "__main__":
    simulation = CitySimulation()
    simulation.command_loop()


# In[ ]:
