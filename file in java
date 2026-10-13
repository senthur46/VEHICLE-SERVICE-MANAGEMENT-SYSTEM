import java.sql.*;
import java.time.LocalDate;
import java.util.*;
// Vehicle Service Management System - JDBC + MySQL version (complete program in a single file)
// Custom checked exception for invalid user input / business-rule violations
class InvalidInputException extends Exception {
public InvalidInputException(String message) {
super(message);
}
}
// Custom checked exception thrown when a registration number is not found
class VehicleNotFoundException extends Exception {
public VehicleNotFoundException(String regNo) {
super("No vehicle registered with number: " + regNo);
}
}
// Interface: anything that can be billed must provide calculateBill()
interface Billable {
double GST_RATE = 0.18; // 18% GST on service charges
double calculateBill();
}
// Enum of services offered by the service centre with their standard labour charge
enum ServiceType {
GENERAL_SERVICE("General Service", 1000),
OIL_CHANGE("Oil Change", 800),
WHEEL_ALIGNMENT("Wheel Alignment", 600),
BRAKE_REPAIR("Brake Repair", 1200),
WASHING("Washing & Cleaning", 300);
private final String label;
private final double cost;
ServiceType(String label, double cost) {
this.label = label;
this.cost = cost;
}
public String getLabel() { return label; }
public double getCost() { return cost; }
}
// Life-cycle of a service request
enum Status {
PENDING, IN_PROGRESS, COMPLETED
}
// Abstract base class: common data and behaviour of every vehicle
abstract class Vehicle {
private String regNo;
private String ownerName;
private String phone;
private String model;
public Vehicle(String regNo, String ownerName, String phone, String model) {
this.regNo = regNo;
this.ownerName = ownerName;
this.phone = phone;
this.model = model;
}
public String getRegNo() { return regNo; }
public String getOwnerName() { return ownerName; }
public String getPhone() { return phone; }
public String getModel() { return model; }
public abstract double getBaseCharge(); // each vehicle type has its own base charge
public abstract String getType();
@Override
public String toString() {
return String.format("%-6s | %-12s | %-14s | %-10s | %-10s",
getType(), regNo, model, ownerName, phone);
}
13
}
class Car extends Vehicle {
public Car(String regNo, String ownerName, String phone, String model) {
super(regNo, ownerName, phone, model);
}
@Override
public double getBaseCharge() { return 1500; }
@Override
public String getType() { return "Car"; }
}
class Bike extends Vehicle {
public Bike(String regNo, String ownerName, String phone, String model) {
super(regNo, ownerName, phone, model);
}
@Override
public double getBaseCharge() { return 500; }
@Override
public String getType() { return "Bike"; }
}
// Utility class: all input validation lives here
class Validator {
public static void validateRegNo(String regNo) throws InvalidInputException {
// Format example: TN67AB1234
if (regNo == null || !regNo.matches("[A-Z]{2}\\d{2}[A-Z]{1,2}\\d{4}"))
throw new InvalidInputException("Invalid registration number (expected format TN67AB1234)");
}
public static void validatePhone(String phone) throws InvalidInputException {
if (phone == null || !phone.matches("[6-9]\\d{9}"))
throw new InvalidInputException("Invalid phone number (10 digits, starting with 6-9)");
}
public static void validateName(String name) throws InvalidInputException {
if (name == null || name.trim().isEmpty())
throw new InvalidInputException("Name cannot be empty");
}
public static void validateCost(double cost) throws InvalidInputException {
if (cost < 0)
throw new InvalidInputException("Cost cannot be negative");
}
}
class Truck extends Vehicle {
private double loadTons; // load capacity in tonnes
public Truck(String regNo, String ownerName, String phone, String model, double loadTons) {
super(regNo, ownerName, phone, model);
this.loadTons = loadTons;
}
// Base charge depends on load capacity: Rs.3000 + Rs.200 per tonne
public double getLoadTons() { return loadTons; }
@Override
public double getBaseCharge() { return 3000 + 200 * loadTons; }
@Override
public String getType() { return "Truck"; }
}
// JDBC connection helper: change URL / USER / PASSWORD to match your MySQL setup
class DBConnection {
private static final String URL = "jdbc:mysql://localhost:3306/vehicle_service";
private static final String USER = "root";
private static final String PASSWORD = "root";
public static Connection getConnection() throws SQLException {
return DriverManager.getConnection(URL, USER, PASSWORD);
}
}
// One service job for one vehicle (a row of the service_record table). Implements Billable.
class ServiceRecord implements Billable {
private final int id;
14
private final Vehicle vehicle;
private final ServiceType type;
private final double partsCost;
private final Status status;
private final LocalDate date;
public ServiceRecord(int id, Vehicle vehicle, ServiceType type, double partsCost, Status status, LocalDate date) {
this.id = id;
this.vehicle = vehicle;
this.type = type;
this.partsCost = partsCost;
this.status = status;
this.date = date;
}
public int getId() { return id; }
public Vehicle getVehicle() { return vehicle; }
public ServiceType getType() { return type; }
public Status getStatus() { return status; }
public double getSubTotal() {
return vehicle.getBaseCharge() + type.getCost() + partsCost;
}
@Override
public double calculateBill() {
return getSubTotal() * (1 + GST_RATE);
}
public String printBill() {
StringBuilder sb = new StringBuilder();
sb.append("========== SERVICE BILL ==========\n");
sb.append("Job ID : ").append(id).append("\n");
sb.append("Date : ").append(date).append("\n");
sb.append("Vehicle : ").append(vehicle.getType()).append(" - ").append(vehicle.getRegNo()).append("\n");
sb.append("Owner : ").append(vehicle.getOwnerName()).append("\n");
sb.append("Service : ").append(type.getLabel()).append("\n");
sb.append(String.format("Base charge : Rs.%.2f%n", vehicle.getBaseCharge()));
sb.append(String.format("Labour : Rs.%.2f%n", type.getCost()));
sb.append(String.format("Parts : Rs.%.2f%n", partsCost));
sb.append(String.format("Sub-total : Rs.%.2f%n", getSubTotal()));
sb.append(String.format("GST (18%%) : Rs.%.2f%n", getSubTotal() * GST_RATE));
sb.append(String.format("TOTAL : Rs.%.2f%n", calculateBill()));
sb.append("==================================");
return sb.toString();
}
@Override
public String toString() {
return String.format("%-5d | %-12s | %-18s | %-11s | %s",
id, vehicle.getRegNo(), type.getLabel(), status, date);
}
}
// Core business logic: every operation reads/writes the MySQL database through JDBC
class ServiceCenter {
private static final String RECORD_SQL =
"SELECT r.job_id, r.type_code, r.parts_cost, r.status, r.service_date, "
+ "v.reg_no, v.vehicle_type, v.owner_name, v.phone, v.model, v.load_tons "
+ "FROM service_record r JOIN vehicle v ON r.reg_no = v.reg_no";
// Build the correct Vehicle subclass from a database row
private Vehicle toVehicle(ResultSet rs) throws SQLException {
String type = rs.getString("vehicle_type");
String reg = rs.getString("reg_no"), owner = rs.getString("owner_name");
String phone = rs.getString("phone"), model = rs.getString("model");
if (type.equals("Car")) return new Car(reg, owner, phone, model);
if (type.equals("Bike")) return new Bike(reg, owner, phone, model);
return new Truck(reg, owner, phone, model, rs.getDouble("load_tons"));
}
private ServiceRecord toRecord(ResultSet rs) throws SQLException {
return new ServiceRecord(rs.getInt("job_id"), toVehicle(rs),
ServiceType.valueOf(rs.getString("type_code")), rs.getDouble("parts_cost"),
Status.valueOf(rs.getString("status")), rs.getDate("service_date").toLocalDate());
}
public void registerVehicle(Vehicle v) throws InvalidInputException, SQLException {
Validator.validateRegNo(v.getRegNo());
Validator.validateName(v.getOwnerName());
Validator.validatePhone(v.getPhone());
String sql = "INSERT INTO vehicle (reg_no, vehicle_type, owner_name, phone, model, load_tons) VALUES (?,?,?,?,?,? try (Connection con = DBConnection.getConnection(); PreparedStatement ps = con.prepareStatement(sql)) {
15
ps.setString(1, v.getRegNo());
ps.setString(2, v.getType());
ps.setString(3, v.getOwnerName());
ps.setString(4, v.getPhone());
ps.setString(5, v.getModel());
if (v instanceof Truck) ps.setDouble(6, ((Truck) v).getLoadTons());
else ps.setNull(6, Types.DECIMAL);
ps.executeUpdate();
} catch (SQLIntegrityConstraintViolationException e) {
throw new InvalidInputException("Vehicle already registered: " + v.getRegNo());
}
}
public Vehicle findVehicle(String regNo) throws VehicleNotFoundException, SQLException {
try (Connection con = DBConnection.getConnection();
PreparedStatement ps = con.prepareStatement("SELECT * FROM vehicle WHERE reg_no = ?")) {
ps.setString(1, regNo);
ResultSet rs = ps.executeQuery();
if (!rs.next()) throw new VehicleNotFoundException(regNo);
return toVehicle(rs);
}
}
// Method overloading: with and without spare-parts cost
public ServiceRecord bookService(String regNo, ServiceType type)
throws VehicleNotFoundException, InvalidInputException, SQLException {
return bookService(regNo, type, 0);
}
public ServiceRecord bookService(String regNo, ServiceType type, double parts)
throws VehicleNotFoundException, InvalidInputException, SQLException {
Validator.validateCost(parts);
Vehicle v = findVehicle(regNo);
LocalDate today = LocalDate.now();
String sql = "INSERT INTO service_record (reg_no, type_code, parts_cost, status, service_date) VALUES (?,?,?,?,?) try (Connection con = DBConnection.getConnection();
PreparedStatement ps = con.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
ps.setString(1, regNo);
ps.setString(2, type.name());
ps.setDouble(3, parts);
ps.setString(4, Status.PENDING.name());
ps.setDate(5, java.sql.Date.valueOf(today));
ps.executeUpdate();
ResultSet keys = ps.getGeneratedKeys();
keys.next();
return new ServiceRecord(keys.getInt(1), v, type, parts, Status.PENDING, today);
}
}
private ServiceRecord findRecord(int id) throws InvalidInputException, SQLException {
try (Connection con = DBConnection.getConnection();
PreparedStatement ps = con.prepareStatement(RECORD_SQL + " WHERE r.job_id = ?")) {
ps.setInt(1, id);
ResultSet rs = ps.executeQuery();
if (!rs.next()) throw new InvalidInputException("No service job with ID " + id);
return toRecord(rs);
}
}
public void updateStatus(int id, Status newStatus) throws InvalidInputException, SQLException {
ServiceRecord r = findRecord(id);
if (r.getStatus() == Status.COMPLETED)
throw new InvalidInputException("Job " + id + " is already completed");
try (Connection con = DBConnection.getConnection();
PreparedStatement ps = con.prepareStatement("UPDATE service_record SET status = ? WHERE job_id = ?")) {
ps.setString(1, newStatus.name());
ps.setInt(2, id);
ps.executeUpdate();
}
}
public String generateBill(int id) throws InvalidInputException, SQLException {
ServiceRecord r = findRecord(id);
if (r.getStatus() != Status.COMPLETED)
throw new InvalidInputException("Bill can be generated only for COMPLETED jobs");
return r.printBill();
}
private List<ServiceRecord> getAllRecords() throws SQLException {
List<ServiceRecord> list = new ArrayList<>();
try (Connection con = DBConnection.getConnection();
Statement st = con.createStatement();
ResultSet rs = st.executeQuery(RECORD_SQL + " ORDER BY r.job_id")) {
16
while (rs.next()) list.add(toRecord(rs));
}
return list;
}
public double totalRevenue() throws SQLException {
double total = 0;
for (ServiceRecord r : getAllRecords())
if (r.getStatus() == Status.COMPLETED) total += r.calculateBill();
return total;
}
public void listVehicles() throws SQLException {
try (Connection con = DBConnection.getConnection();
Statement st = con.createStatement();
ResultSet rs = st.executeQuery("SELECT * FROM vehicle ORDER BY reg_no")) {
if (!rs.isBeforeFirst()) { System.out.println("No vehicles registered."); return; }
System.out.println("Type | Reg. No | Model | Owner | Phone");
while (rs.next()) System.out.println(toVehicle(rs));
}
}
public void listRecords() throws SQLException {
List<ServiceRecord> list = getAllRecords();
if (list.isEmpty()) { System.out.println("No service jobs yet."); return; }
System.out.println("JobID | Reg. No | Service | Status | Date");
for (ServiceRecord r : list) System.out.println(r);
}
}
public class Main {
static Scanner sc = new Scanner(System.in);
static ServiceCenter center = new ServiceCenter();
public static void main(String[] args) {
try (Connection con = DBConnection.getConnection()) {
System.out.println("Connected to MySQL database: " + con.getCatalog());
} catch (SQLException e) {
System.out.println("Cannot connect to MySQL: " + e.getMessage());
return;
}
int choice = -1;
while (choice != 0) {
System.out.println("\n===== VEHICLE SERVICE MANAGEMENT SYSTEM =====");
System.out.println("1. Register Vehicle 2. Book Service 3. Update Job Status");
System.out.println("4. Generate Bill 5. View Vehicles 6. View Service Jobs");
System.out.println("7. Total Revenue 0. Exit");
System.out.print("Enter choice: ");
try {
choice = Integer.parseInt(sc.nextLine().trim());
switch (choice) {
case 1: register(); break;
case 2: book(); break;
case 3: update(); break;
case 4:
System.out.print("Job ID: ");
System.out.println(center.generateBill(Integer.parseInt(sc.nextLine().trim())));
break;
case 5: center.listVehicles(); break;
case 6: center.listRecords(); break;
case 7: System.out.printf("Total revenue (completed jobs): Rs.%.2f%n", center.totalRevenue()); break; case 0: System.out.println("Thank you. Goodbye!"); break;
default: System.out.println("Please choose a number between 0 and 7.");
}
} catch (NumberFormatException e) {
System.out.println("Error: please enter a valid number.");
} catch (InvalidInputException | VehicleNotFoundException e) {
System.out.println("Error: " + e.getMessage());
} catch (SQLException e) {
System.out.println("Database error: " + e.getMessage());
} finally {
if (choice != 0) System.out.println(" ------------------------------------------- ");
}
}
}
static String ask(String prompt) {
System.out.print(prompt);
return sc.nextLine().trim();
}
static void register() throws InvalidInputException, SQLException {
System.out.print("Type (1-Car 2-Bike 3-Truck): ");
17
int t = Integer.parseInt(sc.nextLine().trim());
String reg = ask("Registration No: ").toUpperCase();
String owner = ask("Owner name: ");
String phone = ask("Phone: ");
String model = ask("Model: ");
Vehicle v;
if (t == 1) v = new Car(reg, owner, phone, model);
else if (t == 2) v = new Bike(reg, owner, phone, model);
else if (t == 3) v = new Truck(reg, owner, phone, model, Double.parseDouble(ask("Load capacity (tonnes): ")));
else throw new InvalidInputException("Invalid vehicle type");
center.registerVehicle(v);
System.out.println("Vehicle registered successfully.");
}
static void book() throws InvalidInputException, VehicleNotFoundException, SQLException {
String reg = ask("Registration No: ").toUpperCase();
ServiceType[] types = ServiceType.values();
for (int i = 0; i < types.length; i++)
System.out.printf("%d. %s (Rs.%.0f)%n", i + 1, types[i].getLabel(), types[i].getCost());
int s = Integer.parseInt(ask("Select service: "));
if (s < 1 || s > types.length) throw new InvalidInputException("Invalid service number");
double parts = Double.parseDouble(ask("Spare parts cost (0 if none): "));
ServiceRecord r = center.bookService(reg, types[s - 1], parts);
System.out.println("Service booked. Job ID = " + r.getId());
}
static void update() throws InvalidInputException, SQLException {
int id = Integer.parseInt(ask("Job ID: "));
int s = Integer.parseInt(ask("New status (1-PENDING 2-IN_PROGRESS 3-COMPLETED): "));
if (s < 1 || s > 3) throw new InvalidInputException("Invalid status number");
center.updateStatus(id, Status.values()[s - 1]);
System.out.println("Status updated.");
}
}
