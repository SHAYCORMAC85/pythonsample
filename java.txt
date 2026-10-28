import java.util.ArrayList;
import java.util.Calendar;
import java.util.LinkedList;
import java.util.List;
import org.cloudbus.cloudsim.*;
import org.cloudbus.cloudsim.core.CloudSim;
import org.cloudbus.cloudsim.provisioners.BwProvisionerSimple;
import org.cloudbus.cloudsim.provisioners.PeProvisionerSimple;
import org.cloudbus.cloudsim.provisioners.RamProvisionerSimple;
public class Cloudsim {
 /** The cloudlet list. */
 private static List<Cloudlet> cloudletList;
 /** The vmlist. */
 private static List<Vm> vmlist;
 public static void main(String[] args) {
 try {
 // 1. Initialize CloudSim
 int num_user = 1;
 Calendar calendar = Calendar.getInstance();
 boolean trace_flag = false;
 CloudSim.init(num_user, calendar, trace_flag);
 // 2. Create Datacenter
 Datacenter datacenter0 = createDatacenter("Datacenter_SARA");
 // 3. Create Broker
 DatacenterBroker broker = createBroker();
 int brokerId = broker.getId();
 // 4. Create Virtual Machine (VM)
 vmlist = new ArrayList<Vm>();
 int vmid = 0;
 int mips = 1000;
 long size = 10000; // image size (MB)
 int ram = 512; // vm memory (MB)
 long bw = 1000;
 int pesNumber = 1; // number of cpus
 String vmm = "Xen"; // VMM name
Vm vm = new Vm(vmid, brokerId, mips, pesNumber, ram, bw, size, vmm, new
CloudletSchedulerTimeShared());
 vmlist.add(vm);
 broker.submitVmList(vmlist);
 // 5. Create Cloudlet (Task)
 cloudletList = new ArrayList<Cloudlet>();
 int id = 0;
 long length = 400000;
 long fileSize = 300;
 long outputSize = 300;
 UtilizationModel utilizationModel = new UtilizationModelFull();
Cloudlet cloudlet = new Cloudlet(id, length, pesNumber, fileSize, outputSize, utilizationModel,
utilizationModel, utilizationModel);
 cloudlet.setUserId(brokerId);
 cloudletList.add(cloudlet);
 broker.submitCloudletList(cloudletList);
 // 6. Start Simulation
 CloudSim.startSimulation();
 List<Cloudlet> newList = broker.getCloudletReceivedList();
 CloudSim.stopSimulation();
 // 7. Print Output
 printCloudletList(newList);
 System.out.println("End of prgram !!!!");
 } catch (Exception e) {
 e.printStackTrace();
 }
 }
 private static Datacenter createDatacenter(String name) {
 List<Host> hostList = new ArrayList<Host>();
 List<Pe> peList = new ArrayList<Pe>();
 int mips = 1000;
 peList.add(new Pe(0, new PeProvisionerSimple(mips)));
 int hostId = 0;
 int ram = 2048;
 long storage = 1000000;
 int bw = 10000;
hostList.add(new Host(hostId, new RamProvisionerSimple(ram), new BwProvisionerSimple(bw), storage,
peList, new VmSchedulerTimeShared(peList)));
 String arch = "x86";
 String os = "Linux";
 String vmm = "Xen";
 double time_zone = 10.0;
 double cost = 3.0;
 double costPerMem = 0.05;
 double costPerStorage = 0.001;
 double costPerBw = 0.0;
DatacenterCharacteristics characteristics = new DatacenterCharacteristics(arch, os, vmm, hostList,
time_zone, cost, costPerMem, costPerStorage, costPerBw);
 try {
 return new Datacenter(name, characteristics, new VmAllocationPolicySimple(hostList), new
LinkedList<Storage>(), 0);
 } catch (Exception e) {
 return null;
 }
 }
private static DatacenterBroker createBroker() {
 try {
 return new DatacenterBroker("Broker");
 } catch (Exception e) {
 return null;
 }
 }
 private static void printCloudletList(List<Cloudlet> list) {
 String indent = " ";
 System.out.println();
 System.out.println("========== OUTPUT ==========");
System.out.println("Cloudlet ID" + indent + "STATUS" + indent + "Data center ID" + indent + "VM
ID" + indent + "Time" + indent + "Start Time" + indent + "Finish Time");
 for (Cloudlet cloudlet : list) {
 System.out.print(indent + cloudlet.getCloudletId() + indent + indent);
 if (cloudlet.getCloudletStatus() == Cloudlet.SUCCESS) {
 System.out.print("SUCCESS");
System.out.println(indent + indent + cloudlet.getResourceId() + indent + indent + indent +
cloudlet.getVmId() + indent + indent + (int)cloudlet.getActualCPUTime() + indent + indent +
"0.1" + indent + indent + indent + (0.1 + cloudlet.getActualCPUTime()));
 }
 }
 }
}
