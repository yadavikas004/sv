# Server Virtualization (VMware vSphere 6.7) - 10-Mark Q&A Format

This guide provides structured, point-wise answers designed to help you score full points on **10-mark exam questions**. The points are kept brief for easy memorization, but provide enough technical "meat" to satisfy a long-answer grading rubric.

---

### Unit I: Introducing & Installing ESXi/vCenter

**Q: What is VMware ESXi and its Architecture? (Probability: 95%)**
**A:**
*   **Definition:** A bare-metal (Type 1) hypervisor running directly on physical server hardware.
*   **No OS Dependency:** Eliminates the need for a general-purpose host OS (like Windows), reducing the attack footprint.
*   **Core Engine:** Uses the lightweight 'VMkernel' to manage CPU, memory, storage, and networking directly.
*   **Resource Partitioning:** Abstracts and completely isolates physical hardware resources into independent Virtual Machines.
*   **DCUI:** Features a Direct Console User Interface for initial physical configuration (IP addresses, root passwords).
*   **Management:** Managed locally via the HTML5 Host Client or centrally via vCenter Server.
*   **Efficiency:** Extremely minimal disk payload (~150MB), meaning almost 100% of hardware power goes to the VMs.

**Keywords:** Bare-metal, Type 1, VMkernel, DCUI, Isolation.


**Q: Explain vSphere Auto Deploy & vCenter SSO. (Probability: 80%)**
**A:**
*   **Auto Deploy Definition:** A centralized tool that provisions hundreds of ESXi hosts rapidly without local hard drives.
*   **Stateless Operation:** Hosts boot directly from the network and run entirely in RAM.
*   **Core Protocols:** Relies heavily on PXE (Pre-boot Execution Environment), DHCP, and TFTP.
*   **vCenter Server:** The centralized management utility for the entire vSphere datacenter.
*   **Platform Services Controller (PSC):** Manages shared infrastructure services for vCenter.
*   **Single Sign-On (SSO):** An internal security broker (typically the `vsphere.local` domain).
*   **Single Login Token:** Allows an admin to log in once securely, exchanging tokens without repeated password prompts to manage multiple ESXi hosts.

**Keywords:** Network Streaming, PXE/TFTP, Stateless Boot, PSC, vsphere.local.

---

### Unit II: Networking, Storage, HA & Security

**Q: Contrast Virtual Switches with Physical Switches. (Probability: 85%)**
**A:**
*   **Software-Based:** A vSwitch operates entirely in hypervisor RAM, unlike hardware-based physical switches.
*   **MAC Learning:** Physical switches learn MAC addresses from passing traffic; vSwitches already know the exact MACs of their registered VMs.
*   **Up-links:** vSwitches use physical network cards (vmnics) as "uplinks" to reach the physical world.
*   **No STP Required:** vSwitches are engineered to prevent network loops by default, so they do NOT use Spanning Tree Protocol (STP).
*   **Port Groups:** vSwitches logically separate traffic using Port Groups and VLAN tags.
*   **No Cascading:** You cannot connect two vSwitches directly to each other via virtual cables.

**Keywords:** Software-Based, No STP, Port Groups, Physical Uplinks, Hypervisor-Level.


**Q: What is a VMkernel port and its uses? (Probability: 95%)**
**A:**
*   **Definition:** A logically dedicated routing interface for the ESXi host’s own system traffic (not standard VM traffic).
*   **Management:** Used by administrators to access and configure the ESXi host via the Host Client or vCenter.
*   **vMotion Traffic:** Provides the dedicated bandwidth required to securely live-migrate VMs between hosts.
*   **IP Storage:** Used to connect the host to network-attached storage architectures like iSCSI or NFS.
*   **Fault Tolerance:** Handles the intense real-time synchronization logging required for vSphere FT.
*   **vSAN Traffic:** Handles the backend cluster communication for VMware vSAN nodes.

**Keywords:** System Traffic, vMotion port, IP Storage, Host Management Interface.


**Q: Compare Shared vs. Local Storage and define vSAN. (Probability: 90%)**
**A:**
*   **Local Storage:** Hard drives attached directly inside the physical ESXi chassis (DAS - Direct Attached Storage).
*   **Limitation of Local:** VMs on local storage cannot use High Availability (HA) or vMotion because other physical hosts can't access those drives.
*   **Shared Storage:** A central network storage array (SAN or NAS) accessible by all ESXi hosts simultaneously.
*   **Advanced Features:** Shared storage is a mandatory requirement for vMotion, fault tolerance, and HA.
*   **vSAN Definition:** VMware’s proprietary Software-Defined Storage (SDS) solution.
*   **vSAN Mechanism:** It logically pools together the *local* disks (SSDs/HDDs) of multiple ESXi hosts over the network.
*   **vSAN Benefit:** Creates a highly resilient, unified shared datastore without the massive capital cost of an external SAN.

**Keywords:** Shared Datastore, SAN/NAS, vMotion Requirement, Software-Defined Storage (vSAN).

---

### Unit III: Managing VMs and Resource Allocation

**Q: Differentiate between a Clone and a Template. (Probability: 90%)**
**A:**
*   **Clone Definition:** An exact, fully functional duplicate of a live or powered-off virtual machine.
*   **Clone Purpose:** Used for troubleshooting, creating backups, or safely testing patches on a specific VM before touching production.
*   **Template Definition:** A locked, "golden master" image that cannot be powered on or accidentally altered.
*   **Template Purpose:** Used as a pristine baseline to repeatedly deploy completely identical, standardized VMs.
*   **Customization Specs:** Templates use "Customization Specifications" to inject new computer names, IPs, and SIDs during automated deployment.
*   **Updates:** To update a template with OS patches, you must convert it back to a VM, patch the VM, and convert it to a template again.

**Keywords:** Golden Master, Standardization, Customization Specification, Exact Copy.


**Q: Explain Memory Transparent Page Sharing (TPS) and Memory Reclamation. (Probability: 95%)**
**A:**
*   **Memory Overcommitment:** ESXi hypervisors allow assigning more total RAM to VMs than physically exists on the motherboards.
*   **TPS Definition:** A memory deduplication technique built heavily into the VMkernel.
*   **How TPS Works:** It scans the physical RAM to find identical memory blocks requested by multiple VMs (like core Windows OS `.dll` files).
*   **Storage Saving:** It drops all duplicate copies, keeps one master copy in physical RAM, and points all VMs safely to it.
*   **Ballooning (Level 2):** If physical RAM gets tight, the hypervisor forces the guest OS to use its own paging file to return RAM to the host.
*   **Compression (Level 3):** Automatically zips up memory pages to take up significantly less physical space.
*   **Swapping (Level 4):** The absolute last resort; dumping live VM memory to the physical disk (causing severe lag).

**Keywords:** Memory Overcommitment, Deduplication, Ballooning, Compression, Swapping.

---

### Unit IV: Balancing Utilization & Performance

**Q: Explain vMotion and Storage vMotion features. (Probability: 100%)**
**A:**
*   **vMotion Definition:** The zero-downtime, live migration of a running VM's compute boundary from one ESXi host to another.
*   **Zero Loss:** Client-server connections to the VM remain completely active, usually without dropping a single packet.
*   **vMotion Requirements:** Strictly requires shared storage, matching CPU architecture (or EVC mode), and a Gigabit VMkernel network.
*   **Memory Pre-copy:** vMotion transfers the VM's active memory bits over the network invisibly before shifting the CPU lock.
*   **Storage vMotion:** The live migration of a VM's disk files (.vmdk) across different datastores (e.g., migrating off an expiring SAN array).
*   **Execution:** Storage vMotion happens while the VM is fully turned on, migrating the storage blocks sequentially in the background.

**Keywords:** Live Migration, Zero Downtime, Shared Storage Requirement, Datastore Transfer.


**Q: What is Cross-vCenter vMotion and PowerCLI? (Probability: 85%)**
**A:**
*   **Cross-vCenter vMotion:** A highly advanced iteration of the standard vMotion technology.
*   **Scope:** Allows live migration of VMs across different clusters, distinct architectural datacenters, and fundamentally different vCenter boundary domains.
*   **Use Case:** Painlessly migrating enterprise workloads from an on-premise datacenter directly into a cloud datacenter.
*   **PowerCLI Definition:** A vastly adopted command-line infrastructure tool provided by VMware.
*   **Technology Foundation:** Built natively on top of Microsoft PowerShell framework.
*   **Automation:** Contains hundreds of custom cmdlets to automate mundane vSphere tasks programmatically (e.g., scripting the creation of 500 VMs simultaneously).

**Keywords:** Multi-vCenter Migration, Scripting, PowerShell Cmdlets, Programmable Automation.

---

### The Generic "Fallback" Paragraph for 10-Mark Questions
*(If you forget a specific 10-mark answer but know the topic revolves around virtualization, writing this 5-point breakdown will almost guarantee you partial-to-high marks by demonstrating strong architectural understanding of the syllabus themes.)*

*   **Hardware Abstraction:** ESXi hypervisors seamlessly decouple the physical hardware (CPU, RAM, Storage, Network) from the operating system layer.
*   **Utmost Isolation:** Multiple Virtual Machines execute side-by-side but remain totally unaware of each other, guaranteeing absolute security and crash boundaries.
*   **Optimum Utilization:** Transforms legacy physical servers running at an abysmal 10% CPU load into highly efficient host machines operating gracefully at 80%+ capacity.
*   **High Availability (HA):** Centralized vCenter structures allow VMs to instantly and automatically reboot on surviving network servers if a physical motherboard catastrophically crashes.
*   **Workload Mobility:** Advanced features like vMotion permit critical workloads to shift seamlessly across the datacenter without experiencing client downtime.
*   **Green Datacenter Logistics:** Virtualization drastically cuts down the physical server rack footprint, shrinking electricity consumption and physical cooling needs, which directly reduces capital and operational expenditure (CapEx/OpEx).
