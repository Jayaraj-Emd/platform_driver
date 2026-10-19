LINUX PLATFORM DRIVER
=====================


1. PLATFORM DRIVER
------------------

Purpose:

    A platform driver is a Linux device driver used mainly
    for devices that are directly integrated into a SoC/system.

Examples:

    UART
    GPIO controller
    I2C controller
    SPI controller
    Ethernet controller
    Watchdog
    Timer
    SoC-specific peripherals


Typical Platform Driver Flow:

    Power ON
        |
        v
    Boot ROM
        |
        v
    Bootloader (U-Boot)
        |
        v
    Linux Kernel starts
        |
        v
    Device Tree / DTB is available
        |
        v
    Device Tree is processed
        |
        v
    platform_device is created
        |
        v
    platform_driver is registered
        |
        v
    Driver Core / Platform Bus performs matching
        |
        v
    Match successful
        |
        v
    probe() is called
        |
        v
    Driver gets resources
        |
        v
    Driver initializes hardware


Important:

    probe() does NOT mean physical power-on.

    Hardware may already be powered enough for Linux to access it.

    However, the driver may still need to configure:

        - Clocks
        - Resets
        - Regulators
        - Power domains
        - Pin control
        - Interrupts
        - MMIO registers
        - DMA
        - PHY


------------------------------------------------------------


2. DEVICE DRIVER MATCHING
-------------------------

Question:

    Who performs device-driver matching?


Answer:

    The Linux Driver Core / Platform Bus infrastructure
    performs device-driver matching.

The driver provides information that tells Linux:

    "I can handle this hardware."


Conceptual Flow:

    Device Tree
         |
         | compatible
         v
    platform_device
         |
         | matching
         v
    platform_driver
         |
         | of_match_table
         v
    probe()


Important:

    The driver does not manually search every platform device.

    The Linux driver core performs the matching.


------------------------------------------------------------


3. DEVICE TREE
--------------

Purpose:

    Device Tree describes hardware to the Linux kernel.

Example:

    ethernet@40000000 {
        compatible = "infineon,traveo-ethernet";
        reg = <0x40000000 0x1000>;
        interrupts = <42>;
    };


Important properties:

    compatible
        Describes hardware compatibility.

    reg
        Describes the device address/resource range.

    interrupts
        Describes interrupt resources.


Conceptual Flow:

    Device Tree
         |
         v
    Hardware description
         |
         v
    Linux creates device representation
         |
         v
    Driver matching


------------------------------------------------------------


4. compatible
-------------

Purpose:

    Identifies which hardware a driver is compatible with.

Example:

    compatible = "infineon,traveo-ethernet";


Driver:

    static const struct of_device_id my_match[] = {
        {
            .compatible = "infineon,traveo-ethernet",
        },
        { }
    };


Important:

    The compatible string in Device Tree is matched against
    the driver's of_match_table.


Conceptual Flow:

    Device Tree:

        compatible = "infineon,traveo-ethernet";

                    |
                    v

    Driver:

        .of_match_table = my_match

                    |
                    v

                Matching

                    |
                    v

                  probe()


Interview Point:

    compatible is a Device Tree property used to describe
    hardware compatibility and is commonly used for
    Device Tree based driver matching.


------------------------------------------------------------


5. struct of_device_id
----------------------

Purpose:

    struct of_device_id represents an entry in a Device Tree
    matching table.


Example:

    static const struct of_device_id my_match[] = {
        {
            .compatible = "infineon,traveo-ethernet",
        },
        { }
    };


Important Field:

    .compatible

        Contains the compatible string that can match
        a Device Tree node.


Terminating Entry:

    { }


Purpose of terminating entry:

    Marks the end of the matching table.


Important:

    struct of_device_id itself does not perform matching.

    It provides matching information to the Linux
    Device Tree / Driver Core infrastructure.


------------------------------------------------------------


6. of_match_table
------------------

Purpose:

    of_match_table connects a platform driver with
    its Device Tree matching table.


Example:

    static const struct of_device_id my_match[] = {
        {
            .compatible = "infineon,traveo-ethernet",
        },
        { }
    };


    static struct platform_driver my_driver = {
        .probe = my_probe,

        .driver = {
            .name = "my_eth",
            .of_match_table = my_match,
        },
    };


Flow:

    Device Tree compatible
            |
            v
    of_match_table
            |
            v
    Driver matching
            |
            v
    probe()


Interview Point:

    of_match_table contains the Device Tree compatible
    information used by the driver matching mechanism.


------------------------------------------------------------


7. WHAT HAPPENS IF of_match_table IS MISSING?
----------------------------------------------

Question:

    What happens if a platform driver does not have
    an of_match_table?


Answer:

    If the device is described by Device Tree and the driver
    relies on normal OF/compatible matching, the driver cannot
    bind through that OF matching path.


Important:

    This does NOT mean probe() can never be called.

    Platform devices can also be matched using other
    platform matching mechanisms.


Interview Answer:

    of_match_table is required for normal Device Tree
    compatible-based matching.

    Without it, the OF matching path cannot match that
    driver using its compatible string.


------------------------------------------------------------


8. .driver.name VS compatible
-----------------------------

These two are NOT the same thing.


Example Device Tree:

    compatible = "infineon,traveo-ethernet";


Example Driver:

    .driver = {
        .name = "my_eth",
        .of_match_table = my_match,
    };


.driver.name:

    Represents the Linux driver's name/identity.


compatible:

    Describes the hardware compatibility.


Important:

    .driver.name does NOT have to be the same as
    the Device Tree compatible string.


Interview Point:

    .driver.name identifies the Linux driver.

    compatible identifies the hardware compatibility.


------------------------------------------------------------


9. MULTIPLE compatible STRINGS
------------------------------

A Device Tree node can contain multiple compatible strings.


Example:

    compatible = "vendor,new-device",
                 "vendor,device-family";


Purpose:

    The first string can describe the most specific hardware.

    The second string can describe a more generic
    compatible hardware family.


Concept:

    Specific hardware
          |
          v
    Generic hardware family


Benefit:

    Allows a driver to support multiple hardware generations
    or variants.


------------------------------------------------------------


10. .data IN struct of_device_id
---------------------------------

Purpose:

    .data can associate driver-specific data with a
    particular compatible entry.


Example:

    static const struct of_device_id my_match[] = {
        {
            .compatible = "vendor,device-a",
            .data = &device_a_data,
        },

        {
            .compatible = "vendor,device-b",
            .data = &device_b_data,
        },

        { }
    };


Useful when:

    Multiple hardware variants use the same driver but
    have differences such as:

        - Register layout
        - Features
        - Capabilities
        - Hardware quirks
        - Configuration


Concept:

    compatible
        |
        v
    Match specific hardware
        |
        v
    Associated .data
        |
        v
    Driver uses variant-specific information


------------------------------------------------------------


11. struct platform_driver
--------------------------

Purpose:

    Represents a platform driver in the Linux kernel.


Basic Structure:

    static struct platform_driver my_driver = {
        .probe = my_probe,

        .driver = {
            .name = "my_driver",
            .of_match_table = my_match,
        },
    };


Important members:

    .probe

        Called when the driver successfully matches
        with a platform device.


    .driver.name

        Driver name/identity.


    .driver.of_match_table

        Device Tree matching table.


------------------------------------------------------------


12. probe()
-----------

Purpose:

    probe() is the main initialization callback of a
    platform driver.

It is called after the Linux driver core successfully
matches a device with the driver.


Prototype:

    int probe(struct platform_device *pdev);


Typical implementation:

    static int my_probe(struct platform_device *pdev)
    {
        /* initialize hardware */

        return 0;
    }


Argument:

    pdev

        Type:

            struct platform_device *pdev

        Represents the specific platform device instance
        that matched the driver.


The driver can use pdev to obtain:

    - Resources
    - Device information
    - Device Tree information
    - IRQs
    - Platform-specific data


Return Value:

    0

        Probe successful.


    Negative error code

        Probe failed.


Examples:

    return -ENODEV;
    return -ENOMEM;
    return -EBUSY;


Interview Point:

    probe() is called after successful device-driver matching
    and is normally where the driver obtains resources and
    initializes the hardware.


------------------------------------------------------------


13. platform_driver_register()
------------------------------

Purpose:

    Registers a platform driver with the Linux Driver Core.


Prototype:

    int platform_driver_register(
        struct platform_driver *drv
    );


Argument:

    drv

        Pointer to the platform_driver structure.


Example:

    platform_driver_register(&my_driver);


Return Value:

    0

        Registration successful.


    Negative error value

        Registration failed.


Why is it required?

    Linux needs to know that the driver exists.

    Registration makes the driver available to the
    Driver Core for device-driver matching.


Concept:

    Driver code
         |
         v
    platform_driver
         |
         | register
         v
    Linux Driver Core
         |
         v
    Device matching


------------------------------------------------------------


14. platform_driver_unregister()
--------------------------------

Purpose:

    Unregisters a platform driver from the Linux Driver Core.


Prototype:

    void platform_driver_unregister(
        struct platform_driver *drv
    );


Argument:

    drv

        Pointer to the platform_driver structure.


Example:

    platform_driver_unregister(&my_driver);


Return Value:

    void


Why is it required?

    It removes the driver from the platform driver's
    registration with the Driver Core.


------------------------------------------------------------


15. module_platform_driver()
----------------------------

Purpose:

    Convenience macro used to register and unregister
    a platform driver.


Without the macro:

    static int __init my_init(void)
    {
        return platform_driver_register(&my_driver);
    }


    static void __exit my_exit(void)
    {
        platform_driver_unregister(&my_driver);
    }


    module_init(my_init);
    module_exit(my_exit);


Using the macro:

    module_platform_driver(my_driver);


Important:

    module_platform_driver() is a convenience mechanism.

    It does not replace the concepts of:

        platform_driver_register()

        platform_driver_unregister()


Interview Point:

    Understand the underlying register/unregister APIs first.

    module_platform_driver() simply reduces boilerplate.


------------------------------------------------------------


16. PLATFORM RESOURCE
---------------------

A platform device can contain resources such as:

    - MMIO memory
    - IRQ
    - Other platform-specific resources


Example Device Tree:

    ethernet@40000000 {
        compatible = "infineon,traveo-ethernet";
        reg = <0x40000000 0x1000>;
        interrupts = <42>;
    };


Conceptual flow:

    Device Tree
         |
         v
    platform_device
         |
         v
    Resources
         |
         v
    Driver retrieves resources


------------------------------------------------------------


17. platform_get_resource()
---------------------------

Purpose:

    Retrieves a resource associated with a platform device.


Prototype:

    struct resource *platform_get_resource(
        struct platform_device *pdev,
        unsigned int type,
        unsigned int num
    );


Arguments:


    pdev

        Platform device passed to probe().


    type

        Type of resource being requested.

        Common examples:

            IORESOURCE_MEM

                Memory/MMIO resource.


            IORESOURCE_IRQ

                Interrupt resource.


    num

        Zero-based index of the resource.


Example:

    struct resource *res;

    res = platform_get_resource(
        pdev,
        IORESOURCE_MEM,
        0
    );


Return Value:

    Success:

        struct resource *


    Failure:

        NULL


Example with error handling:

    struct resource *res;

    res = platform_get_resource(
        pdev,
        IORESOURCE_MEM,
        0
    );

    if (!res)
        return -ENODEV;


Important:

    platform_get_resource() only retrieves the resource.

    It does NOT:

        - Reserve the resource
        - Map the resource
        - Access the hardware


Interview Point:

    platform_get_resource() retrieves a resource from
    the platform device.


------------------------------------------------------------


18. RESOURCE INDEXING
---------------------

Resources are zero-indexed.


Example Device Tree:

    reg = <0x40000 0x1000>,
          <0x50000 0x1000>;


This represents two memory resources.


First resource:

    index = 0

    start = 0x40000

    size = 0x1000


Second resource:

    index = 1

    start = 0x50000

    size = 0x1000


Driver:

    platform_get_resource(
        pdev,
        IORESOURCE_MEM,
        0
    );


Gets:

    First memory resource.


Driver:

    platform_get_resource(
        pdev,
        IORESOURCE_MEM,
        1
    );


Gets:

    Second memory resource.


Important:

    Resource index starts from 0.


------------------------------------------------------------


19. struct resource
-------------------

Purpose:

    Represents a resource range associated with a device.


Important fields:

    start

        Starting address of the resource.


    end

        Ending address of the resource.


    flags

        Describes the resource type/properties.


Conceptual structure:

    struct resource {
        resource_size_t start;
        resource_size_t end;
        unsigned long flags;
        ...
    };


Example:

    start = 0x40000
    end   = 0x40FFF
    flags = IORESOURCE_MEM


------------------------------------------------------------


20. resource->start
-------------------

Purpose:

    Represents the starting address of a resource.


Example:

    res->start


If:

    res->start = 0x40000000


Then:

    Resource starts at:

        0x40000000


For an MMIO resource, this represents the beginning
of the MMIO address range.


------------------------------------------------------------


21. resource->end
-----------------

Purpose:

    Represents the last address of the resource range.


Example:

    start = 0x40000
    size  = 0x1000


Then:

    end = start + size - 1


Therefore:

    end = 0x40000 + 0x1000 - 1

    end = 0x40FFF


Resource range:

    0x40000
        |
        |
        v
    0x40FFF


------------------------------------------------------------


22. RESOURCE SIZE
-----------------

Formula:

    size = end - start + 1


Example:

    start = 0x40000
    end   = 0x40FFF


    size = 0x40FFF - 0x40000 + 1

         = 0x1000


Therefore:

    Resource size = 0x1000 bytes


Important:

    The +1 is required because both start and end
    addresses are inclusive.


------------------------------------------------------------


23. resource_size()
-------------------

Purpose:

    Returns the size of a struct resource.


Conceptually:

    resource_size(res)


Equivalent conceptually to:

    res->end - res->start + 1


Example:

    struct resource *res;

    res = platform_get_resource(
        pdev,
        IORESOURCE_MEM,
        0
    );

    if (!res)
        return -ENODEV;

    size = resource_size(res);


Return Value:

    Size of the resource.


Important:

    resource_size() does NOT:

        - Allocate memory
        - Reserve the resource
        - Map the resource
        - Access hardware


It only calculates the resource size.


Interview Point:

    resource_size() is preferred over manually calculating:

        res->end - res->start + 1


------------------------------------------------------------


24. resource_size_t
-------------------

Purpose:

    resource_size_t is used by the kernel for resource
    addresses and sizes.


Why?

    Resource addresses can be wider than 32 bits
    on some systems.


Important:

    Do not casually store hardware resource addresses
    or sizes in an int.


Concept:

    Hardware resource address
            |
            v
    resource_size_t


------------------------------------------------------------


25. request_mem_region()
------------------------

Purpose:

    Reserve/claim an I/O memory resource range.


Prototype:

    struct resource *request_mem_region(
        resource_size_t start,
        resource_size_t n,
        const char *name
    );


Arguments:


    start

        Starting address of the memory resource.


    n

        Number of bytes to reserve.


    name

        Name of the resource owner.


Return Value:

    Success:

        Returns struct resource *


    Failure:

        Returns NULL if the requested resource range
        cannot be claimed.


Example:

    struct resource *mem;

    mem = request_mem_region(
        res->start,
        resource_size(res),
        "my_ethernet"
    );

    if (!mem)
        return -EBUSY;


Purpose:

    Prevents multiple users/drivers from simultaneously
    claiming the same I/O memory range.


Important:

    request_mem_region() only reserves/claims
    the address range.

    It does NOT map the physical address into
    kernel virtual address space.


Mapping is a separate operation:

    ioremap()


Interview Point:

    platform_get_resource()
        -> Retrieve resource

    resource_size()
        -> Calculate size

    request_mem_region()
        -> Reserve/claim resource

    ioremap()
        -> Map resource

    readl()/writel()
        -> Access registers


------------------------------------------------------------


26. RETRIEVE VS RESERVE VS MAP VS ACCESS
----------------------------------------

These operations are different.


platform_get_resource():

    Retrieve/find the resource.


resource_size():

    Calculate resource size.


request_mem_region():

    Reserve/claim the resource.


ioremap():

    Map the MMIO address into kernel virtual address space.


readl()/writel():

    Access the hardware registers.


Mental model:

    Retrieve
       |
       v
    Calculate size
       |
       v
    Reserve
       |
       v
    Map
       |
       v
    Access


------------------------------------------------------------


27. Device Tree reg
-------------------

Example:

    reg = <0x40000000 0x1000>;


Conceptually:

    Start address:

        0x40000000


    Size:

        0x1000


    End:

        0x40000FFF


Therefore:

    Address range:

        0x40000000
        to
        0x40000FFF


Important:

    Device Tree reg describes an address range in the
    address space of the parent bus.


On a simple SoC:

    The bus address commonly corresponds directly
    to the CPU physical MMIO address.


More precise statement:

    Device Tree reg contains a bus address.

    The parent bus may translate this address before
    it reaches the CPU/device address space.


Interview Answer:

    "reg describes the device resource address and size
    in the parent bus address space. On a simple SoC it
    commonly corresponds directly to the physical MMIO
    address."


------------------------------------------------------------


28. PHYSICAL ADDRESS VS LOGICAL ADDRESS
---------------------------------------

Example:

    reg = <0x40000000 0x1000>;


This is NOT a normal kernel virtual/logical address.


It describes:

    Hardware / bus address


Conceptually:

    Device Tree
        |
        | reg
        v
    Bus / Physical MMIO address
        |
        v
    Resource
        |
        v
    ioremap()
        |
        v
    Kernel Virtual Address


Important:

    Linux uses virtual memory.

    A hardware physical/bus address should not normally
    be directly used as a normal C pointer.


------------------------------------------------------------


29. WHY NOT DIRECTLY CAST MMIO ADDRESS?
----------------------------------------

Do NOT normally do:

    volatile u32 *reg = (u32 *)0x40000000;


Why?

    Linux uses virtual memory.

    Physical/bus addresses are not automatically
    valid kernel virtual pointers.


Instead:

    Physical / bus address
            |
            v
         ioremap()
            |
            v
    Kernel virtual MMIO address
            |
            v
       readl()/writel()


------------------------------------------------------------


30. ioremap()
-------------

Purpose:

    Maps an MMIO physical/bus address range into
    kernel virtual address space.


Conceptual Prototype:

    void __iomem *ioremap(
        resource_size_t offset,
        unsigned long size
    );


Arguments:


    offset

        Physical/bus address of the MMIO region.


    size

        Number of bytes to map.


Example:

    void __iomem *base;

    base = ioremap(
        res->start,
        resource_size(res)
    );

    if (!base)
        return -ENOMEM;


Return Value:

    A __iomem pointer representing the mapped
    MMIO region.

    Failure:

        NULL


Important:

    ioremap() performs mapping.

    It does NOT reserve the resource.

    Reservation and mapping are separate concepts.


------------------------------------------------------------


31. __iomem
-----------

Purpose:

    __iomem is a kernel annotation used for pointers
    that refer to I/O memory.


Example:

    void __iomem *base;


Important:

    A __iomem pointer should be accessed using the
    appropriate MMIO accessors.


Use:

    readl()
    writel()
    readb()
    writeb()
    readw()
    writew()


Do not treat it like an ordinary RAM pointer.


------------------------------------------------------------


32. iounmap()
------------

Purpose:

    Removes a mapping previously created by ioremap()
    when using the non-managed mapping approach.


Conceptual Prototype:

    void iounmap(void __iomem *addr);


Argument:

    addr

        MMIO virtual address returned by ioremap().


Example:

    iounmap(base);


Return Value:

    void


Important:

    iounmap() removes the virtual mapping.

    It does not mean that a hardware resource has
    been reserved or released in every other sense.


------------------------------------------------------------


33. MMIO
--------

MMIO means:

    Memory-Mapped I/O


Concept:

    Hardware registers are mapped into an address space
    that the CPU can access using memory-style addresses.


Example:

    Ethernet controller:

        Base address = 0x40000000


Registers:

    CONTROL register:

        offset = 0x00


    STATUS register:

        offset = 0x04


Then:

    CONTROL address:

        base + 0x00


    STATUS address:

        base + 0x04


Concept:

    Hardware base
        +
    Register offset
        =
    Register address


------------------------------------------------------------


34. MMIO REGISTER ACCESS
------------------------

After mapping:

    void __iomem *base;


Register access is performed using MMIO accessors.


Read:

    value = readl(base + offset);


Write:

    writel(value, base + offset);


Important:

    Use kernel MMIO accessors instead of treating
    MMIO as normal RAM.


------------------------------------------------------------


35. readl()
----------

Purpose:

    Reads a 32-bit value from an MMIO register.


Conceptual Prototype:

    u32 readl(const volatile void __iomem *addr);


Argument:

    addr

        Address of the MMIO register.


Example:

    u32 status;

    status = readl(base + STATUS_REG);


Return Value:

    32-bit register value.


Important:

    readl() is for MMIO register access.

    It should not be used for normal RAM.


------------------------------------------------------------


36. writel()
------------

Purpose:

    Writes a 32-bit value to an MMIO register.


Conceptual Prototype:

    void writel(
        u32 value,
        volatile void __iomem *addr
    );


Arguments:


    value

        Value to write to the register.


    addr

        MMIO register address.


Example:

    writel(0x1, base + CONTROL_REG);


Return Value:

    void


Important:

    writel() is used for MMIO register access.


------------------------------------------------------------


37. readb(), readw(), readl(), readq()
--------------------------------------

These APIs read different register widths.


readb():

    Read 8-bit value.


readw():

    Read 16-bit value.


readl():

    Read 32-bit value.


readq():

    Read 64-bit value.


Concept:

    readb()
        -> 8 bits

    readw()
        -> 16 bits

    readl()
        -> 32 bits

    readq()
        -> 64 bits


The register width must match the hardware specification.


------------------------------------------------------------


38. writeb(), writew(), writel(), writeq()
------------------------------------------

These APIs write different register widths.


writeb():

    Write 8-bit value.


writew():

    Write 16-bit value.


writel():

    Write 32-bit value.


writeq():

    Write 64-bit value.


Important:

    Always follow the hardware register specification
    when selecting the access width.


------------------------------------------------------------


39. COMPLETE MMIO FLOW
----------------------

Device Tree:

    ethernet@40000000 {
        compatible = "infineon,traveo-ethernet";
        reg = <0x40000000 0x1000>;
    };


Step 1:

    Driver matches Device Tree compatible.


Step 2:

    probe() is called.


Step 3:

    Retrieve resource:

    res = platform_get_resource(
        pdev,
        IORESOURCE_MEM,
        0
    );


Step 4:

    Check resource:

    if (!res)
        return -ENODEV;


Step 5:

    Calculate size:

    size = resource_size(res);


Step 6:

    Reserve resource:

    mem = request_mem_region(
        res->start,
        size,
        "my_ethernet"
    );


Step 7:

    Map resource:

    base = ioremap(
        res->start,
        size
    );


Step 8:

    Access registers:

    status = readl(base + STATUS_REG);


Step 9:

    Write registers:

    writel(value, base + CONTROL_REG);


Conceptual flow:

    Device Tree
         |
         v
    compatible
         |
         v
    Driver matching
         |
         v
    probe()
         |
         v
    platform_get_resource()
         |
         v
    struct resource
         |
         +---- start
         +---- end
         +---- flags
         |
         v
    resource_size()
         |
         v
    request_mem_region()
         |
         v
    ioremap()
         |
         v
    void __iomem *base
         |
         v
    readl()/writel()
         |
         v
    Hardware registers


------------------------------------------------------------


40. COMPLETE PLATFORM DRIVER SKELETON
-------------------------------------

Example:

    #include <linux/module.h>
    #include <linux/platform_device.h>
    #include <linux/of.h>
    #include <linux/io.h>

    static const struct of_device_id my_match[] = {
        {
            .compatible = "infineon,traveo-ethernet",
        },
        { }
    };

    static int my_probe(struct platform_device *pdev)
    {
        struct resource *res;

        res = platform_get_resource(
            pdev,
            IORESOURCE_MEM,
            0
        );

        if (!res)
            return -ENODEV;

        /*
         * MMIO mapping and register initialization
         * will be performed here.
         */

        return 0;
    }

    static struct platform_driver my_driver = {
        .probe = my_probe,

        .driver = {
            .name = "my_eth",
            .of_match_table = my_match,
        },
    };

    module_platform_driver(my_driver);

    MODULE_LICENSE("GPL");
    MODULE_AUTHOR("Jayaraj");
    MODULE_DESCRIPTION("Platform Driver Learning Example");


------------------------------------------------------------


41. API SUMMARY
---------------

platform_driver_register()

    Purpose:

        Register platform driver.


    Return:

        0 / negative error.


------------------------------------------------------------

platform_driver_unregister()

    Purpose:

        Unregister platform driver.


    Return:

        void.


------------------------------------------------------------

module_platform_driver()

    Purpose:

        Convenience macro for platform driver
        registration/unregistration.


    Return:

        Not a normal function return value.


------------------------------------------------------------

platform_get_resource()

    Purpose:

        Retrieve platform resource.


    Return:

        struct resource * / NULL.


------------------------------------------------------------

resource_size()

    Purpose:

        Calculate resource size.


    Return:

        Resource size.


------------------------------------------------------------

request_mem_region()

    Purpose:

        Reserve/claim I/O memory resource.


    Return:

        struct resource * / NULL.


------------------------------------------------------------

ioremap()

    Purpose:

        Map MMIO address into kernel virtual address space.


    Return:

        __iomem mapped address / NULL on failure.


------------------------------------------------------------

iounmap()

    Purpose:

        Remove an MMIO mapping.


    Return:

        void.


------------------------------------------------------------

readl()

    Purpose:

        Read 32-bit MMIO register.


    Return:

        u32.


------------------------------------------------------------

writel()

    Purpose:

        Write 32-bit MMIO register.


    Return:

        void.


------------------------------------------------------------


42. MOST IMPORTANT DIFFERENCE
-----------------------------

Remember these four operations:


    platform_get_resource()

        "Where is my resource?"


    resource_size()

        "How big is my resource?"


    request_mem_region()

        "Can I claim this resource?"


    ioremap()

        "How do I access this MMIO resource
         from kernel virtual address space?"


Then:


    readl()/writel()

        "Read/write the hardware register."


------------------------------------------------------------


43. INTERVIEW QUICK REVISION
----------------------------

Q1. What is a platform driver?

Answer:

    A Linux driver used mainly for SoC-integrated devices
    represented as platform devices.


Q2. Who performs device-driver matching?

Answer:

    Linux Driver Core / Platform Bus infrastructure.


Q3. What is compatible?

Answer:

    A Device Tree property describing hardware compatibility
    and commonly used for OF-based driver matching.


Q4. What is of_match_table?

Answer:

    A driver's table containing Device Tree matching entries.


Q5. What is struct of_device_id?

Answer:

    A structure representing an entry in the Device Tree
    matching table.


Q6. What is probe()?

Answer:

    The driver callback called after successful
    device-driver matching.


Q7. What does platform_driver_register() do?

Answer:

    Registers a platform driver with the Linux Driver Core.


Q8. What does platform_get_resource() do?

Answer:

    Retrieves a resource associated with a platform device.


Q9. Does platform_get_resource() map the resource?

Answer:

    No.


Q10. What does resource_size() do?

Answer:

    Calculates the size of a resource.


Q11. What does request_mem_region() do?

Answer:

    Claims/reserves an I/O memory address range.


Q12. Does request_mem_region() map the address?

Answer:

    No.


Q13. What does ioremap() do?

Answer:

    Maps an MMIO physical/bus address range into
    kernel virtual address space.


Q14. Why can't we directly cast a physical address?

Answer:

    Linux uses virtual memory, so a physical/bus MMIO
    address is not automatically a valid kernel pointer.


Q15. What is __iomem?

Answer:

    A kernel annotation for pointers referring to
    I/O memory.


Q16. What does readl() do?

Answer:

    Reads a 32-bit MMIO register.


Q17. What does writel() do?

Answer:

    Writes a 32-bit MMIO register.


Q18. What is the difference between .driver.name
    and compatible?

Answer:

    .driver.name identifies the Linux driver.

    compatible describes the hardware compatibility.


------------------------------------------------------------


44. COMPLETE MENTAL MODEL
-------------------------

Keep this flow in mind:


    HARDWARE DESCRIPTION
            |
            v
       Device Tree
            |
            |
       +----+----+
       |         |
       v         v
    compatible   reg
       |         |
       |         v
       |      Resource
       |         |
       +----+----+
            |
            v
       platform_device
            |
            v
       Driver Matching
            |
            |
       of_match_table
            |
            v
       platform_driver
            |
            v
          probe()
            |
            v
    platform_get_resource()
            |
            v
       struct resource
            |
       +----+----+
       |    |    |
       v    v    v
     start end flags
            |
            v
      resource_size()
            |
            v
    request_mem_region()
            |
            v
         ioremap()
            |
            v
     void __iomem *base
            |
            v
      readl()/writel()
            |
            v
      HARDWARE REGISTERS


------------------------------------------------------------


45. TOPICS COMPLETED SO FAR
---------------------------

Platform Driver:

    [x] What is a platform driver?
    [x] Platform device concept
    [x] Platform driver concept
    [x] Boot-to-probe flow
    [x] Driver Core
    [x] Platform bus
    [x] Device-driver matching
    [x] probe()
    [x] platform_driver_register()
    [x] platform_driver_unregister()
    [x] module_platform_driver()


Device Tree:

    [x] Device Tree purpose
    [x] compatible
    [x] reg
    [x] interrupts concept
    [x] struct of_device_id
    [x] of_match_table
    [x] Missing of_match_table
    [x] .driver.name vs compatible
    [x] Multiple compatible strings
    [x] .data


Resources:

    [x] Platform resources
    [x] platform_get_resource()
    [x] struct resource
    [x] resource->start
    [x] resource->end
    [x] resource->flags
    [x] Resource indexing
    [x] resource_size()
    [x] resource_size_t
    [x] request_mem_region()


MMIO:

    [x] MMIO concept
    [x] Device Tree reg
    [x] Bus/physical address concept
    [x] Physical vs kernel virtual address
    [x] Why direct pointer casting is wrong
    [x] ioremap() concept
    [x] __iomem concept
    [x] iounmap() concept
    [x] readl()
    [x] writel()
    [x] readb/readw/readl/readq
    [x] writeb/writew/writel/writeq


------------------------------------------------------------


46. NEXT TOPICS
---------------

The next topics to learn in order:


    1. ioremap() in deeper detail

    2. iounmap()

    3. readl()

    4. writel()

    5. readb()
       readw()
       readl()
       readq()

    6. writeb()
       writew()
       writel()
       writeq()

    7. devm_ioremap_resource()

    8. Managed vs non-managed APIs

    9. Error handling and cleanup

    10. Device Tree resource parsing

    11. Platform driver hands-on

    12. Device Tree overlay / custom DTS

    13. Clocks

    14. Reset controller

    15. GPIO

    16. IRQ from Device Tree

    17. I2C

    18. SPI


Important Learning Rule:

    Learn the normal/non-managed APIs first.

    Then learn the devm_* convenience APIs.

    Do not memorize devm APIs without understanding
    what they are managing internally.


------------------------------------------------------------


47. API LEARNING METHOD
-----------------------

For every new Linux kernel API, learn:


    1. What is it?

    2. Why is it required?

    3. Prototype

    4. Return value

    5. Every important argument

    6. Alternative APIs

    7. Example

    8. Real driver usage

    9. Common mistakes

    10. Interview questions


Goal:

    Understand the API and the underlying kernel concept.

    Do not simply memorize code.


------------------------------------------------------------


END OF CURRENT PLATFORM DRIVER NOTES
------------------------------------

Current Status:

    Platform Driver + Device Tree + Resource Handling
    + MMIO fundamentals completed.


Next:

    Detailed MMIO APIs and hands-on implementation.

------------------------------------------------------------

48. READ-MODIFY-WRITE
----------------------

Purpose:

    A common MMIO operation where the driver:

        1. Reads a register
        2. Modifies selected bits
        3. Writes the value back

Example:

    value = readl(base + CONTROL_REG);
    value |= BIT(0);
    writel(value, base + CONTROL_REG);


Important:

    Read-modify-write is NOT automatically safe for every
    hardware register.

Why can it be dangerous?

    Hardware may modify register bits asynchronously between
    the read and write.

    Other reasons include:

        - Write-1-to-clear (W1C) bits
        - Read-only bits
        - Write-only bits
        - Read side effects
        - Reserved bits
        - Hardware changing bits asynchronously


Interview Point:

    Do not blindly use read-modify-write on an MMIO register.

    First check the hardware register specification.


------------------------------------------------------------

49. HARDWARE SET/CLEAR REGISTERS
--------------------------------

Some peripherals provide special hardware registers such as:

    CONTROL_REG
    CONTROL_SET
    CONTROL_CLEAR


Example:

    writel(BIT(2), base + CONTROL_SET);


Meaning:

    BIT(2) = 0000 0100

    Writing this to CONTROL_SET tells the hardware to set
    bit 2 in CONTROL_REG.


Similarly:

    writel(BIT(2), base + CONTROL_CLEAR);

    tells the hardware to clear bit 2.


Important:

    CONTROL_SET and CONTROL_CLEAR are NOT Linux APIs.

    They are hardware-specific registers provided by the
    peripheral.


Why are they useful?

    They can allow a driver to change a particular bit without
    reading the entire register and writing an old copy back.


Example:

    CONTROL_REG = 1010 0001

    Driver wants to set bit 2.

    writel(BIT(2), base + CONTROL_SET);

    Hardware can produce:

    CONTROL_REG = 1010 0101


Important:

    Not every peripheral provides SET/CLEAR registers.

    Never invent REG_SET or REG_CLEAR addresses.

    The driver must follow the peripheral datasheet.


------------------------------------------------------------

50. MMIO ACCESS WIDTH
---------------------

Hardware registers can have different access widths.

Example:

    REGISTER        OFFSET      WIDTH
    ---------------------------------
    STATUS          0x00        8-bit
    CONTROL         0x04        16-bit
    CONFIG          0x08        32-bit
    COUNTER         0x10        64-bit


Read APIs:

    readb()  -> 8-bit
    readw()  -> 16-bit
    readl()  -> 32-bit
    readq()  -> 64-bit


Write APIs:

    writeb() -> 8-bit
    writew() -> 16-bit
    writel() -> 32-bit
    writeq() -> 64-bit


Important:

    The hardware register specification determines the
    required access width.


------------------------------------------------------------

51. readb()
-----------

Purpose:

    Reads an 8-bit value from an MMIO register.


Conceptual Prototype:

    u8 readb(const volatile void __iomem *addr);


Argument:

    addr

        Address of the MMIO register.


Example:

    u8 status;

    status = readb(base + STATUS_REG);


Return Value:

    u8


Width:

    8 bits = 1 byte


Interview Point:

    Use readb() when the hardware register specification
    requires an 8-bit MMIO access.


------------------------------------------------------------

52. readw()
-----------

Purpose:

    Reads a 16-bit value from an MMIO register.


Conceptual Prototype:

    u16 readw(const volatile void __iomem *addr);


Argument:

    addr

        Address of the MMIO register.


Example:

    u16 status;

    status = readw(base + STATUS_REG);


Return Value:

    u16


Width:

    16 bits = 2 bytes


Interview Point:

    Use readw() when the hardware register specification
    requires a 16-bit MMIO access.


------------------------------------------------------------

53. readl()
-----------

Purpose:

    Reads a 32-bit value from an MMIO register.


Conceptual Prototype:

    u32 readl(const volatile void __iomem *addr);


Argument:

    addr

        Address of the MMIO register.


Example:

    u32 status;

    status = readl(base + STATUS_REG);


Return Value:

    u32


Width:

    32 bits = 4 bytes


Important:

    readl() is for MMIO register access, not normal RAM.


------------------------------------------------------------

54. readq()
-----------

Purpose:

    Reads a 64-bit value from an MMIO register.


Conceptual Prototype:

    u64 readq(const volatile void __iomem *addr);


Argument:

    addr

        Address of the MMIO register.


Example:

    u64 counter;

    counter = readq(base + COUNTER_REG);


Return Value:

    u64


Width:

    64 bits = 8 bytes


------------------------------------------------------------

55. writeb()
------------

Purpose:

    Writes an 8-bit value to an MMIO register.


Conceptual Prototype:

    void writeb(u8 value, volatile void __iomem *addr);


Arguments:

    value

        8-bit value to write.

    addr

        MMIO register address.


Example:

    writeb(value, base + STATUS_REG);


Return Value:

    void


------------------------------------------------------------

56. writew()
------------

Purpose:

    Writes a 16-bit value to an MMIO register.


Conceptual Prototype:

    void writew(u16 value, volatile void __iomem *addr);


Arguments:

    value

        16-bit value to write.

    addr

        MMIO register address.


Example:

    writew(value, base + CONTROL_REG);


Return Value:

    void


------------------------------------------------------------

57. writel()
------------

Purpose:

    Writes a 32-bit value to an MMIO register.


Conceptual Prototype:

    void writel(u32 value, volatile void __iomem *addr);


Arguments:

    value

        32-bit value to write.

    addr

        MMIO register address.


Example:

    writel(0x1, base + CONTROL_REG);


Return Value:

    void


Concept:

    readl():

        Hardware register
            |
            v
          CPU


    writel():

        CPU
         |
         v
        Hardware register


------------------------------------------------------------

58. writeq()
------------

Purpose:

    Writes a 64-bit value to an MMIO register.


Conceptual Prototype:

    void writeq(u64 value, volatile void __iomem *addr);


Arguments:

    value

        64-bit value to write.

    addr

        MMIO register address.


Example:

    writeq(value, base + COUNTER_REG);


Return Value:

    void


------------------------------------------------------------

59. REGISTER WIDTH VS REGISTER SPACING
--------------------------------------

Do NOT assume that register width determines the offset
of the next register.


Example:

    STATUS      0x00      8-bit
    CONTROL     0x04     16-bit
    CONFIG      0x08     32-bit


STATUS is only 8 bits wide, but the next register starts
at offset 0x04.


Therefore:

    Register width
        and
    Register offset/spacing

are separate properties defined by the hardware register map.


Important:

    Always use the offsets and access widths specified by
    the hardware documentation.


------------------------------------------------------------

60. WHY NOT USE readl() FOR EVERY REGISTER?
--------------------------------------------

Suppose:

    STATUS      0x00      8-bit
    CONTROL     0x04     16-bit
    CONFIG      0x08     32-bit


Correct:

    readb(base + 0x00);

    readw(base + 0x04);

    readl(base + 0x08);


Do not simply assume:

    readl(base + 0x00);

for every register.


Reason:

    readl() requests a 32-bit MMIO access.

    If the hardware requires an 8-bit access, using a
    32-bit access may produce incorrect or hardware-specific
    behavior.


It is not guaranteed that readl() simply reads the register
plus the next register.


Interview Answer:

    "I select the MMIO accessor based on the access width
    specified by the peripheral's register specification."


------------------------------------------------------------

61. MMIO WIDTH QUICK REVISION
-----------------------------

    b = byte  = 8-bit
    w = word  = 16-bit
    l = long  = 32-bit
    q = quad  = 64-bit


Read:

    readb()  -> 8-bit
    readw()  -> 16-bit
    readl()  -> 32-bit
    readq()  -> 64-bit


Write:

    writeb() -> 8-bit
    writew() -> 16-bit
    writel() -> 32-bit
    writeq() -> 64-bit


Important:

    The hardware register specification determines the
    required access width.

    Do not choose the accessor only because of the C variable
    type.


------------------------------------------------------------

62. MMIO REGISTER CHECKLIST
---------------------------

Before accessing a hardware register, understand:

    1. Register address/offset
    2. Register width
    3. Read/write permissions
    4. Bit meanings
    5. Special behaviors
    6. Reserved bits
    7. Whether read has side effects
    8. Whether write has special semantics
    9. Whether SET/CLEAR registers exist
    10. Required Linux MMIO accessor


Example:

    CONTROL_REG 0x00

    Bit 0     ENABLE       R/W
    Bit 1     RESET        R/W
    Bit 2     RX_INT       R/W
    Bit 3     RESERVED
    Bit 4     ERROR        W1C
    Bits 5-31 RESERVED


A driver developer should study this information before
writing register access code.


------------------------------------------------------------

63. MMIO ACCESS MENTAL MODEL
----------------------------

    Device Tree
         |
         v
    Resource address
         |
         v
    platform_get_resource()
         |
         v
    struct resource
         |
         v
    ioremap()
         |
         v
    void __iomem *base
         |
         v
    base + register offset
         |
         +----------------------+
         |                      |
         v                      v
      read*()                 write*()
         |                      |
         v                      v
      CPU <- Hardware       CPU -> Hardware


Access width is selected according to the hardware
register specification.


------------------------------------------------------------

64. API SUMMARY - MMIO ACCESSORS
--------------------------------

readb()

    Purpose:
        Read 8-bit MMIO register.

    Return:
        u8


readw()

    Purpose:
        Read 16-bit MMIO register.

    Return:
        u16


readl()

    Purpose:
        Read 32-bit MMIO register.

    Return:
        u32


readq()

    Purpose:
        Read 64-bit MMIO register.

    Return:
        u64


writeb()

    Purpose:
        Write 8-bit MMIO register.

    Return:
        void


writew()

    Purpose:
        Write 16-bit MMIO register.

    Return:
        void


writel()

    Purpose:
        Write 32-bit MMIO register.

    Return:
        void


writeq()

    Purpose:
        Write 64-bit MMIO register.

    Return:
        void


------------------------------------------------------------

65. INTERVIEW QUICK REVISION - MMIO
-----------------------------------

Q1. What does readl() do?

Answer:

    Reads a 32-bit value from an MMIO register.


Q2. What does writel() do?

Answer:

    Writes a 32-bit value to an MMIO register.


Q3. What is the difference between readw() and readl()?

Answer:

    readw() performs a 16-bit read.

    readl() performs a 32-bit read.


Q4. How do you choose between readb/readw/readl/readq?

Answer:

    Based on the access width required by the hardware
    register specification.


Q5. What does writeq() do?

Answer:

    Writes a 64-bit value to an MMIO register.


Q6. Can I use readl() for every register?

Answer:

    No. The required MMIO access width is hardware-specific.


Q7. What is read-modify-write?

Answer:

    Read a register, modify selected bits, and write the
    modified value back.


Q8. Is read-modify-write always safe?

Answer:

    No. It can be unsafe for registers with W1C bits,
    read side effects, asynchronously changing bits,
    reserved bits, and other special hardware semantics.


Q9. What is REG_SET?

Answer:

    It is an example of a hardware-specific set register.
    It is not a Linux API.


Q10. Can every peripheral use REG_SET?

Answer:

    No. Only if the hardware provides such a register.


------------------------------------------------------------

66. TOPICS COMPLETED SO FAR - UPDATED
-------------------------------------

Platform Driver:

    [x] What is a platform driver?
    [x] Platform device concept
    [x] Platform driver concept
    [x] Boot-to-probe flow
    [x] Driver Core
    [x] Platform bus
    [x] Device-driver matching
    [x] probe()
    [x] platform_driver_register()
    [x] platform_driver_unregister()
    [x] module_platform_driver()


Device Tree:

    [x] Device Tree purpose
    [x] compatible
    [x] reg
    [x] interrupts concept
    [x] struct of_device_id
    [x] of_match_table
    [x] Missing of_match_table
    [x] .driver.name vs compatible
    [x] Multiple compatible strings
    [x] .data


Resources:

    [x] Platform resources
    [x] platform_get_resource()
    [x] struct resource
    [x] resource->start
    [x] resource->end
    [x] resource->flags
    [x] Resource indexing
    [x] resource_size()
    [x] resource_size_t
    [x] request_mem_region()


MMIO:

    [x] MMIO concept
    [x] Device Tree reg
    [x] Bus/physical address concept
    [x] Physical vs kernel virtual address
    [x] Why direct pointer casting is wrong
    [x] ioremap() concept
    [x] __iomem concept
    [x] iounmap() concept
    [x] readl()
    [x] writel()
    [x] Read-modify-write concept
    [x] Hardware SET/CLEAR register concept
    [x] readb()
    [x] readw()
    [x] readq()
    [x] writeb()
    [x] writew()
    [x] writeq()
    [x] Register width vs register spacing


------------------------------------------------------------

67. NEXT TOPICS
---------------

Continue in this order:

    1. MMIO register semantics in more detail
       - R/O
       - R/W
       - W/O
       - W1C
       - W1S
       - Reserved bits

    2. MMIO ordering and barriers

    3. Deeper ioremap()/MMIO behavior

    4. devm_ioremap_resource()

    5. Managed vs non-managed APIs

    6. Error handling and cleanup

    7. Device Tree resource parsing

    8. Platform driver hands-on

    9. Device Tree overlay / custom DTS

    10. Clocks

    11. Reset controller

    12. GPIO

    13. IRQ from Device Tree

    14. I2C

    15. SPI


Important Learning Rule:

    Learn the normal/non-managed APIs first.

    Then learn the devm_* convenience APIs.

    Do not memorize devm APIs without understanding
    what they are managing internally.


------------------------------------------------------------

68. API LEARNING METHOD
-----------------------

For every new Linux kernel API, learn:

    1. What is it?
    2. Why is it required?
    3. Prototype
    4. Return value
    5. Every important argument
    6. Alternative APIs
    7. Example
    8. Real driver usage
    9. Common mistakes
    10. Interview questions


Goal:

    Understand the API and the underlying kernel concept.

    Do not simply memorize code.


------------------------------------------------------------

END OF UPDATED PLATFORM DRIVER NOTES
-------------------------------------

Current Status:

    Platform Driver + Device Tree + Resource Handling
    + MMIO fundamentals
    + MMIO access widths
    + Read-modify-write
    + Hardware SET/CLEAR register concept

Next:

    MMIO register semantics and ordering/barriers.

------------------------------------------------------------


69. MMIO REGISTER SEMANTICS
---------------------------

Hardware registers do not always behave like normal RAM.

Before reading or writing a register, the driver must understand
the access type and the meaning of each bit.

Common register semantics:

    R/O
        Read Only

    R/W
        Read / Write

    W/O
        Write Only

    W1C
        Write 1 to Clear

    W1S
        Write 1 to Set

    Reserved
        Not intended for driver use


------------------------------------------------------------


70. R/O — READ ONLY
-------------------

Meaning:

    The register/field is intended to be read by the driver.

Example:

    STATUS register

        Bit 0 = READY
        Bit 1 = ERROR

    value = readl(base + STATUS);

Important:

    Do not write to an R/O field unless the hardware
    specification explicitly defines such behavior.

Interview Point:

    R/O means the field has a defined meaning for reading,
    but the driver should not modify it.


------------------------------------------------------------


71. R/W — READ / WRITE
----------------------

Meaning:

    The driver can read and write the register/field.

Example:

    CONTROL register

        Bit 0 = ENABLE

    value = readl(base + CONTROL);
    value |= BIT(0);
    writel(value, base + CONTROL);

Important:

    For normal R/W registers, read-modify-write can be
    appropriate when other fields must be preserved.

    Always check the hardware specification because a
    register may contain fields with different semantics.


------------------------------------------------------------


72. W/O — WRITE ONLY
--------------------

Meaning:

    The register/field is intended for writes.

Example:

    COMMAND register

        Bit 0 = START
        Bit 1 = STOP

    writel(BIT(0), base + COMMAND);

Important:

    Do not rely on readl() to determine the current value
    of a W/O register.

    A read may return an undefined, meaningless, fixed,
    or otherwise hardware-specific value.

Interview Point:

    W/O means the driver should treat the register as
    write-only and should not depend on its read value.


------------------------------------------------------------


73. W1C — WRITE 1 TO CLEAR
--------------------------

Meaning:

    Writing 1 to a bit clears that bit.

    Writing 0 leaves that bit unchanged.

Example:

    IRQ_STATUS

        Bit 0 = RX_DONE   W1C
        Bit 1 = TX_DONE   W1C
        Bit 2 = ERROR     W1C

Suppose:

    IRQ_STATUS = 0000 0101

To clear only bit 0:

    writel(BIT(0), base + IRQ_STATUS);

Written value:

    0000 0001

Hardware interprets:

    Bit 0 = 1
        -> clear bit 0

    Bit 1 = 0
        -> no change

    Bit 2 = 0
        -> no change

Result:

    0000 0100

Important:

    For W1C, the value written is a command,
    not the new register value.


------------------------------------------------------------


74. W1C AND READ-MODIFY-WRITE HAZARD
------------------------------------

Do not blindly use normal RMW logic on a W1C register.

Example:

    value = readl(base + IRQ_STATUS);
    value |= BIT(0);
    writel(value, base + IRQ_STATUS);

Suppose the register contains:

    0000 0101

After the read:

    value = 0000 0101

After:

    value |= BIT(0);

The value is still:

    0000 0101

Now that value is written to a W1C register.

The hardware interprets:

    bit 0 = 1
        -> clear bit 0

    bit 2 = 1
        -> clear bit 2

Therefore both bit 0 and bit 2 can be cleared.

This is different from a normal R/W register.

For a normal R/W register:

    writing 0000 0101
        -> store that value

For a W1C register:

    writing 0000 0101
        -> clear bit 0 and bit 2

Key rule:

    For W1C, 1 is an action/command.

Also:

    Hardware may update the register between the read
    and write, creating an additional race with hardware.

Preferred operation when clearing one W1C bit:

    writel(BIT(n), base + REGISTER);


------------------------------------------------------------


75. W1S — WRITE 1 TO SET
------------------------

Meaning:

    Writing 1 to a bit sets that bit.

    Writing 0 leaves that bit unchanged.

Example:

    CONTROL_SET

        Bit 0 = ENABLE
        Bit 1 = START
        Bit 2 = IRQ_ENABLE

To set bit 2:

    writel(BIT(2), base + CONTROL_SET);

Written value:

    0000 0100

Hardware interprets:

    Bit 2 = 1
        -> set bit 2

    Other bits = 0
        -> no change

Important:

    Writing 0000 0100 does NOT mean that all zero bits
    become one.

    Only the bit written as 1 is set.

For W1S:

    Write 1 -> set
    Write 0 -> no change


------------------------------------------------------------


76. W1S VS NORMAL READ-MODIFY-WRITE
-----------------------------------

Suppose the hardware provides a W1S register.

Instead of:

    value = readl(REG);
    value |= BIT(2);
    writel(value, REG);

The driver can perform:

    writel(BIT(2), REG_SET);

This allows the hardware to perform the individual
bit-set operation without the driver constructing a
complete register value.

Important:

    W1S semantics are hardware-specific.

    The driver must confirm the register behavior in
    the hardware specification.


------------------------------------------------------------


77. RESERVED BITS
-----------------

Meaning:

    Reserved bits are not defined for the driver's use.

Example:

    CONTROL register

    Bit 0     -> EN       R/W
    Bits 1-3  -> Reserved
    Bits 4-7  -> MODE     R/W
    Bits 8-31 -> Reserved

Important:

    Do not intentionally modify reserved bits unless the
    hardware specification explicitly says it is safe.

Do not blindly write:

    writel(0xFFFFFFFF, base + CONTROL);

because this writes 1 to reserved bits as well.

The behavior of reserved bits is hardware-specific and
should not be assumed.


------------------------------------------------------------


78. RESERVED DOES NOT MEAN READ ONLY
------------------------------------

These concepts are different.

R/O:

    The field has a defined meaning.

    The driver can read it.

    The driver should not write it.

Reserved:

    The field is not defined for the driver's use.

    The driver should not interpret or intentionally
    modify it.

A register may contain both:

    Bit 7 -> STATUS, R/O
    Bit 6 -> Reserved

The driver can still read the whole register:

    value = readl(base + REG);

But it should not interpret the reserved bit as a
meaningful hardware status.


------------------------------------------------------------


79. R/W REGISTER AND RESERVED BITS
----------------------------------

Consider:

    Bit 0 = EN       R/W
    Bit 1 = Reserved
    Bits 2-3 = Reserved
    Bits 4-7 = MODE  R/W
    Bits 8-31 = Reserved

A blind write such as:

    writel(0xFFFFFFFF, base + REG);

is unsafe because it writes to fields that the driver
does not own.

Also, blindly doing:

    writel(BIT(0), base + REG);

may overwrite the other R/W fields such as MODE with zero.

For a normal R/W register, the driver may need to preserve
other fields, for example:

    value = readl(base + REG);
    value |= BIT(0);
    writel(value, base + REG);

However, the exact approach depends on the register semantics
specified by the hardware documentation.


------------------------------------------------------------


80. MMIO REGISTER SEMANTICS — QUICK REVISION
--------------------------------------------

    R/O
        Read only.

    R/W
        Read and write.

    W/O
        Write only. Do not rely on read value.

    W1C
        Write 1 -> clear.
        Write 0 -> no change.

    W1S
        Write 1 -> set.
        Write 0 -> no change.

    Reserved
        Not for driver use.

Most important distinction:

    R/W:
        Written value represents register state.

    W1C:
        Written 1 represents a clear command.

    W1S:
        Written 1 represents a set command.


------------------------------------------------------------


81. INTERVIEW QUICK REVISION — REGISTER SEMANTICS
------------------------------------------------

Q1. What does W1C mean?

Answer:

    Write 1 to Clear.

    Writing 1 clears the corresponding bit.
    Writing 0 leaves the bit unchanged.


Q2. Why can normal RMW be dangerous on W1C?

Answer:

    Because bits read as 1 are written back as 1,
    and in a W1C register each written 1 is a clear
    command. This can clear multiple status bits
    unintentionally.


Q3. What does W1S mean?

Answer:

    Write 1 to Set.

    Writing 1 sets the corresponding bit.
    Writing 0 leaves the bit unchanged.


Q4. Can a W/O register be reliably read?

Answer:

    No. The driver should not rely on the read value
    because the hardware does not guarantee a meaningful
    read result.


Q5. Can an R/O register be read with readl()?

Answer:

    Yes, if the register width is 32 bits and the hardware
    specification defines it accordingly.


Q6. Does Reserved mean Read Only?

Answer:

    No.

    R/O is an access type for a defined field.

    Reserved means the field is not defined for the
    driver's use.


Q7. Why should reserved bits not be blindly written?

Answer:

    Their behavior is not defined for normal driver use.
    Writing them may produce hardware-specific or
    unintended behavior.


------------------------------------------------------------


82. CURRENT MMIO REGISTER SEMANTICS MENTAL MODEL
------------------------------------------------

Before accessing a register:

    1. Find the register address/offset.

    2. Check the register width.

    3. Check each field's access type.

    4. Identify R/O, R/W, W/O, W1C, W1S and Reserved fields.

    5. Understand whether writing a value represents
       register state or a hardware command.

    6. Avoid blind writes.

    7. Avoid normal RMW on special registers such as W1C
       unless the hardware specification explicitly
       supports the operation.

    8. Follow the hardware programming manual.

Mental model:

    Address
       +
    Width
       +
    Register semantics
       +
    Bit-field meaning
       =
    Correct MMIO access


------------------------------------------------------------


83. TOPICS COMPLETED SO FAR — UPDATED
-------------------------------------

Platform Driver:

    [x] What is a platform driver?
    [x] Platform device concept
    [x] Platform driver concept
    [x] Boot-to-probe flow
    [x] Driver Core
    [x] Platform bus
    [x] Device-driver matching
    [x] probe()
    [x] platform_driver_register()
    [x] platform_driver_unregister()
    [x] module_platform_driver()


Device Tree:

    [x] Device Tree purpose
    [x] compatible
    [x] reg
    [x] interrupts concept
    [x] struct of_device_id
    [x] of_match_table
    [x] Missing of_match_table
    [x] .driver.name vs compatible
    [x] Multiple compatible strings
    [x] .data


Resources:

    [x] Platform resources
    [x] platform_get_resource()
    [x] struct resource
    [x] resource->start
    [x] resource->end
    [x] resource->flags
    [x] Resource indexing
    [x] resource_size()
    [x] resource_size_t
    [x] request_mem_region()


MMIO:

    [x] MMIO concept
    [x] Device Tree reg
    [x] Bus/physical address concept
    [x] Physical vs kernel virtual address
    [x] Why direct pointer casting is wrong
    [x] ioremap() concept
    [x] __iomem concept
    [x] iounmap() concept
    [x] readl()
    [x] writel()
    [x] readb/readw/readl/readq
    [x] writeb/writew/writel/writeq
    [x] Read-modify-write
    [x] Hardware SET/CLEAR registers
    [x] Register width vs register spacing
    [x] R/O register semantics
    [x] R/W register semantics
    [x] W/O register semantics
    [x] W1C register semantics
    [x] W1S register semantics
    [x] Reserved bits
    [x] W1C RMW hazard
    [x] W1S individual bit set
    [x] Reserved vs R/O distinction


------------------------------------------------------------


84. NEXT TOPICS
---------------

Continue in this order:

    1. MMIO ordering and memory barriers

    2. Deeper ioremap() / MMIO behavior

    3. devm_ioremap_resource()

    4. Managed vs non-managed APIs

    5. Error handling and cleanup

    6. Device Tree resource parsing

    7. Platform driver hands-on

    8. Device Tree overlay / custom DTS

    9. Clocks

    10. Reset controller

    11. GPIO

    12. IRQ from Device Tree

    13. I2C

    14. SPI


Important Learning Rule:

    Learn normal/non-managed APIs first.

    Then learn devm_* convenience APIs.

    Do not memorize devm APIs without understanding
    what they are managing internally.


------------------------------------------------------------

------------------------------------------------------------

85. DEVM_IOREMAP_RESOURCE()
---------------------------

Purpose:

    devm_ioremap_resource() provides a device-managed way to
    claim/map an MMIO resource.

    It is used after obtaining the MMIO resource from the
    platform device.

Why is it required?

    With manually managed resources, the driver has to manage
    resource lifetime and cleanup explicitly.

Manual model:

    request_mem_region()
        |
        v
    ioremap()
        |
        v
    use MMIO
        |
        v
    iounmap()
        |
        v
    release_mem_region()

Device-managed model:

    devm_ioremap_resource()
        |
        v
    use MMIO
        |
        v
    device/driver detach
        |
        v
    managed cleanup

Important:

    The devm_ prefix means the resource is managed according
    to the device's lifetime.

    This reduces manual cleanup code and makes error paths
    easier to manage.

------------------------------------------------------------

86. SOURCE OF THE RESOURCE
--------------------------

    The MMIO resource ultimately comes from the hardware
    description, such as Device Tree.

    However, devm_ioremap_resource() receives a struct resource *
    obtained from the platform device.

Typical flow:

    Device Tree
        |
        v
    platform device created
        |
        v
    platform_get_resource(pdev, IORESOURCE_MEM, 0)
        |
        v
    struct resource *res
        |
        v
    devm_ioremap_resource(&pdev->dev, res)

Interview Point:

    "I obtain the MMIO struct resource from the platform
    device using platform_get_resource(). The resource itself
    ultimately originates from the hardware description,
    such as Device Tree."

------------------------------------------------------------

87. DEVM_IOREMAP_RESOURCE() API
-------------------------------

Conceptual prototype:

    void __iomem *devm_ioremap_resource(
        struct device *dev,
        const struct resource *res
    );

Arguments:

    dev

        The device whose lifetime manages the resource.

        In a platform driver this is commonly:

            &pdev->dev

    res

        Pointer to the MMIO resource.

        Commonly obtained using:

            res = platform_get_resource(
                pdev,
                IORESOURCE_MEM,
                0
            );

Return value:

    Success:

        void __iomem *

        This is the kernel I/O-memory mapping that can be
        used with MMIO accessors such as readl() and writel().

    Failure:

        Error-encoded pointer.

        Check using:

            IS_ERR(base)

        Obtain the error code using:

            PTR_ERR(base)

Important comparison:

    ioremap()

        Failure -> NULL

    devm_ioremap_resource()

        Failure -> ERR_PTR(...)

Typical pattern:

    struct resource *res;
    void __iomem *base;

    res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    if (!res)
        return -ENODEV;

    base = devm_ioremap_resource(&pdev->dev, res);
    if (IS_ERR(base))
        return PTR_ERR(base);

------------------------------------------------------------

88. ERR_PTR(), IS_ERR(), PTR_ERR()
----------------------------------

    Linux kernel APIs sometimes need to return either:

        - a valid pointer, or
        - a negative error code.

    The kernel can encode the error code inside the pointer.

Conceptually:

    Success:

        base -> valid address

    Failure:

        base -> ERR_PTR(-ENOMEM)

Checking:

    if (IS_ERR(base))
        return PTR_ERR(base);

Meaning:

    IS_ERR(base)

        Checks whether the returned pointer represents
        an error.

    PTR_ERR(base)

        Extracts the original error code.

Interview Point:

    "devm_ioremap_resource() returns an error-encoded pointer
    on failure, so I use IS_ERR() to check it and PTR_ERR()
    to propagate the error."

Do not confuse:

    NULL
        with
    ERR_PTR(...)

    They are different kernel error-handling mechanisms.

------------------------------------------------------------

89. DEVICE-MANAGED MMIO LIFETIME
--------------------------------

Example:

    static int my_probe(struct platform_device *pdev)
    {
        struct resource *res;
        void __iomem *base;

        res = platform_get_resource(
            pdev,
            IORESOURCE_MEM,
            0
        );

        if (!res)
            return -ENODEV;

        base = devm_ioremap_resource(
            &pdev->dev,
            res
        );

        if (IS_ERR(base))
            return PTR_ERR(base);

        value = readl(base + STATUS_REG);

        return 0;
    }

After successful probe:

    The driver can use:

        readl(base + offset)
        writel(value, base + offset)

When the device/driver is detached:

    The device-managed resource is automatically cleaned up.

Therefore, the driver does not explicitly call:

    iounmap(base)

for this devm-managed mapping.

Important:

    devm_ioremap_resource() is not simply an alias for ioremap().

    The important difference is resource lifetime management.

------------------------------------------------------------

90. MANUAL VS DEVICE-MANAGED MMIO
---------------------------------

Manual approach:

    platform_get_resource()
        |
        v
    request_mem_region()
        |
        v
    ioremap()
        |
        v
    readl()/writel()
        |
        v
    iounmap()
        |
        v
    release_mem_region()

Device-managed approach:

    platform_get_resource()
        |
        v
    devm_ioremap_resource()
        |
        v
    readl()/writel()
        |
        v
    device/driver detach
        |
        v
    automatic managed cleanup

Main advantage:

    Less manual cleanup.

    Fewer cleanup paths to maintain.

    Resource lifetime is tied to the device.

------------------------------------------------------------

91. INTERVIEW QUICK REVISION — DEVM_IOREMAP_RESOURCE()
-------------------------------------------------------

Q1. What does devm_ioremap_resource() do?

Answer:

    It provides a device-managed way to map an MMIO resource
    and manage its lifetime.

Q2. Where does the resource come from?

Answer:

    The driver normally obtains the struct resource from
    the platform device using platform_get_resource().

Q3. What does devm_ mean?

Answer:

    Device-managed resource lifetime.

Q4. Do I need to explicitly call iounmap() for a mapping
    created by devm_ioremap_resource()?

Answer:

    No. The mapping is device-managed and is cleaned up
    automatically when the device is detached.

Q5. How do you check the return value?

Answer:

    Use:

        if (IS_ERR(base))

    and obtain the error using:

        PTR_ERR(base)

Q6. What is the difference between ioremap() and
    devm_ioremap_resource()?

Answer:

    ioremap() creates the MMIO mapping and requires the
    driver to manage its lifetime manually.

    devm_ioremap_resource() provides device-managed resource
    handling, reducing the need for manual cleanup.

Q7. What is the important return-value difference?

Answer:

    ioremap():

        failure -> NULL

    devm_ioremap_resource():

        failure -> ERR_PTR(...)

------------------------------------------------------------

92. CURRENT MMIO RESOURCE MENTAL MODEL
--------------------------------------

    Device Tree
        |
        v
    Platform device
        |
        v
    platform_get_resource()
        |
        v
    struct resource
        |
        +-------------------------------+
        |                               |
        v                               v
    Manual API                    Device-managed API
        |                               |
        v                               v
    request_mem_region()       devm_ioremap_resource()
        |                               |
        v                               v
    ioremap()                     void __iomem *base
        |                               |
        +---------------+---------------+
                        |
                        v
                readl()/writel()
                        |
                        v
                    Hardware

Key mental model:

    struct resource
        =
    description of the MMIO resource

    ioremap()
        =
    create a kernel virtual I/O mapping

    devm_ioremap_resource()
        =
    device-managed MMIO resource handling

    readl()/writel()
        =
    access the hardware registers

    iounmap()
        =
    manually remove an ioremap() mapping

------------------------------------------------------------

93. TOPICS COMPLETED SO FAR — UPDATED
-------------------------------------

Platform Driver:

    [x] What is a platform driver?
    [x] Platform device concept
    [x] Platform driver concept
    [x] Boot-to-probe flow
    [x] Driver Core
    [x] Platform bus
    [x] Device-driver matching
    [x] probe()
    [x] platform_driver_register()
    [x] platform_driver_unregister()
    [x] module_platform_driver()

Device Tree:

    [x] Device Tree purpose
    [x] compatible
    [x] reg
    [x] interrupts concept
    [x] struct of_device_id
    [x] of_match_table
    [x] Missing of_match_table
    [x] .driver.name vs compatible
    [x] Multiple compatible strings
    [x] .data

Resources:

    [x] Platform resources
    [x] platform_get_resource()
    [x] struct resource
    [x] resource->start
    [x] resource->end
    [x] resource->flags
    [x] Resource indexing
    [x] resource_size()
    [x] resource_size_t
    [x] request_mem_region()
    [x] Difference between resource claiming and MMIO mapping

MMIO:

    [x] MMIO concept
    [x] Device Tree reg
    [x] Bus/physical address concept
    [x] Physical vs kernel virtual address
    [x] Why direct pointer casting is wrong
    [x] ioremap() concept
    [x] ioremap() mapping size
    [x] ioremap() failure handling
    [x] __iomem concept
    [x] iounmap() concept
    [x] readb/readw/readl/readq
    [x] writeb/writew/writel/writeq
    [x] Read-modify-write
    [x] Hardware SET/CLEAR register concept
    [x] Register width vs register spacing
    [x] R/O register semantics
    [x] R/W register semantics
    [x] W/O register semantics
    [x] W1C register semantics
    [x] W1S register semantics
    [x] Reserved bits
    [x] W1C RMW hazard
    [x] W1S individual bit set
    [x] Reserved vs R/O distinction
    [x] devm_ioremap_resource()
    [x] Device-managed resource lifetime
    [x] IS_ERR()
    [x] PTR_ERR()
    [x] ERR_PTR() concept

------------------------------------------------------------

94. NEXT TOPICS
---------------

Continue in this order:

    1. Managed vs non-managed APIs

    2. Error handling and cleanup

    3. Device Tree resource parsing

    4. Platform driver hands-on

    5. Device Tree overlay / custom DTS

    6. Clocks

    7. Reset controller

    8. GPIO

    9. IRQ from Device Tree

    10. I2C

    11. SPI

Important Learning Rule:

    Learn normal/non-managed APIs first.

    Then learn the devm_* convenience APIs.

    Do not memorize devm APIs without understanding
    what they are managing internally.

------------------------------------------------------------

------------------------------------------------------------


95. MANAGED VS NON-MANAGED RESOURCES — LIFETIME
------------------------------------------------

Non-managed resources:

    The driver explicitly manages the resource lifetime.

    Example:

        ptr = kzalloc(size, GFP_KERNEL);

    The driver must later release the allocation:

        kfree(ptr);


Managed resources:

    The resource is associated with a struct device through
    an appropriate devm_* API.

    The device-managed resource infrastructure records the
    cleanup action and performs cleanup when the managed
    resource lifecycle ends.


Conceptual model:

    struct device
         |
         v
    device-managed resource infrastructure
         |
         +---- managed resource A -> cleanup action A
         |
         +---- managed resource B -> cleanup action B
         |
         +---- managed resource C -> cleanup action C


Important:

    This is a conceptual model. The kernel's implementation
    is more specific than a simple list of resource pointers.


------------------------------------------------------------


96. WHAT HAPPENS WHEN PROBE FAILS?
---------------------------------

Example sequence:

    1. devm_kzalloc() succeeds.
    2. devm_ioremap_resource() succeeds.
    3. A later hardware initialization step fails.
    4. probe() returns an error.

The managed resources acquired for that unsuccessful probe
are cleaned up by the device-managed resource mechanism.

Conceptually:

    probe()
       |
       +--> devm_kzalloc() succeeds
       |
       +--> devm_ioremap_resource() succeeds
       |
       +--> hardware initialization fails
       |
       +--> return error
                 |
                 v
          managed cleanup


Important:

    devm_* does not free the resource as soon as the
    acquisition function returns.

    If probe succeeds, the resource remains available while
    the device is using it.

    If probe fails, managed resources acquired during that
    probe attempt are cleaned up.


------------------------------------------------------------


97. WHAT HAPPENS AFTER SUCCESSFUL PROBE?
---------------------------------------

If probe() returns 0:

    The driver is successfully bound to the device.

    Managed resources remain available while the device is
    using them.

Later, when the driver/device is detached and the managed
resource lifecycle ends, the kernel releases the managed
resources.

Conceptual lifecycle:

    probe()
       |
       v
    acquire managed resources
       |
       v
    probe succeeds
       |
       v
    driver uses resources
       |
       v
    device/driver detached
       |
       v
    managed cleanup


Important:

    devm_* does not mean "free the resource when probe()
    returns."

    It means the resource lifetime is managed in relation
    to the device lifecycle.


------------------------------------------------------------


98. devm_KZALLOC()
------------------

Purpose:

    devm_kzalloc() allocates zero-initialized kernel memory
    and associates the allocation with a struct device's
    managed-resource lifecycle.


Prototype:

    void *devm_kzalloc(struct device *dev,
                       size_t size,
                       gfp_t gfp);


Arguments:

    dev

        The device associated with the allocation.

        In a platform driver, commonly:

            &pdev->dev


    size

        Number of bytes to allocate.

        Example:

            sizeof(*data)


    gfp

        Allocation flags.

        GFP_KERNEL is commonly used when allocation is
        allowed to sleep in normal process context.


Return value:

    Success:

        Valid pointer to zero-initialized memory.


    Failure:

        NULL


Example:

    struct my_device *data;

    data = devm_kzalloc(&pdev->dev,
                        sizeof(*data),
                        GFP_KERNEL);

    if (!data)
        return -ENOMEM;


Important:

    devm_kzalloc() returns NULL on allocation failure.
    The driver may return -ENOMEM to report the failure.

    -ENOMEM is the driver's error return; it is not the
    direct return value of devm_kzalloc().


------------------------------------------------------------


99. WHY ZERO-INITIALIZATION MATTERS
-----------------------------------

Compare:

    kmalloc()

        Allocates memory, but its contents are not
        guaranteed to be zero.


    kzalloc()

        Allocates memory initialized to zero.


Example structure:

    struct my_device {
        int status;
        int irq;
        void __iomem *base;
    };


With kzalloc() or devm_kzalloc(), the allocated bytes are
initialized to zero.

For this example, the fields initially have zero-valued
representations, so the integer fields are zero and the
pointer field is NULL on Linux kernel targets.


Why it helps:

    The driver starts with known initial values rather than
    relying on leftover contents of allocated memory.


Important:

    Zero-initialization does not replace correct
    initialization of hardware, resources, locks, work items,
    or other objects that require dedicated initialization
    APIs.


------------------------------------------------------------


100. kzalloc() VS devm_kzalloc()
--------------------------------

Manual allocation:

    data = kzalloc(sizeof(*data), GFP_KERNEL);

    if (!data)
        return -ENOMEM;

    /* use data */

    /* when appropriate */
    kfree(data);


Device-managed allocation:

    data = devm_kzalloc(&pdev->dev,
                        sizeof(*data),
                        GFP_KERNEL);

    if (!data)
        return -ENOMEM;

    /* use data */


Difference:

    kzalloc():

        The driver is responsible for releasing the memory
        with kfree() at the appropriate point.


    devm_kzalloc():

        The allocation is associated with the device-managed
        resource lifecycle. The driver does not manually
        call kfree() for that allocation.


Both:

    Allocate zero-initialized memory.


------------------------------------------------------------


101. devm_KZALLOC() CLEANUP EXAMPLE
----------------------------------

Suppose probe() does:

    data = devm_kzalloc(&pdev->dev,
                        sizeof(*data),
                        GFP_KERNEL);

    if (!data)
        return -ENOMEM;

    /* later initialization fails */

    return -EIO;


The driver does not need to call kfree(data) for this
device-managed allocation.

The managed-resource mechanism handles cleanup when the
probe attempt fails.


If probe succeeds:

    The allocation remains available while the device uses
    it. It is not freed merely because probe() returned 0.

    Cleanup happens when the relevant device-managed
    lifecycle ends.


------------------------------------------------------------


102. INTERVIEW QUICK REVISION — MANAGED RESOURCES
------------------------------------------------

Q1. What does devm_* mean?

Answer:

    The resource is managed in relation to a struct device's
    lifecycle. The devm API registers the appropriate cleanup
    with the device-managed resource infrastructure.


Q2. Does devm_kzalloc() free memory as soon as probe()
    returns?

Answer:

    No. If probe succeeds, the allocation remains available
    while the device uses it.


Q3. What happens to managed resources if probe fails?

Answer:

    Managed resources acquired for that unsuccessful probe
    are cleaned up by the device-managed resource mechanism.


Q4. What happens when a successfully bound device is
    detached?

Answer:

    The device-managed resource mechanism releases its
    managed resources as the relevant lifecycle ends.


Q5. What does the z in kzalloc() mean?

Answer:

    The allocated memory is initialized to zero.


Q6. What does devm_kzalloc() return on failure?

Answer:

    NULL. The driver commonly returns -ENOMEM.


Q7. Why pass &pdev->dev?

Answer:

    To associate the allocation with the platform device's
    managed-resource lifecycle.


Q8. Do we call kfree() for memory allocated by
    devm_kzalloc()?

Answer:

    No. The corresponding managed cleanup is automatic.


------------------------------------------------------------

105. devm_kmalloc() VS devm_kzalloc()
-----------------------------------

Both APIs allocate memory associated with a device-managed
resource lifecycle.

Prototype:

    void *devm_kmalloc(struct device *dev,
                       size_t size,
                       gfp_t gfp);

    void *devm_kzalloc(struct device *dev,
                       size_t size,
                       gfp_t gfp);

Key difference:

    devm_kmalloc()
        Allocates memory without zero-initializing it.

    devm_kzalloc()
        Allocates memory and initializes the allocated bytes to zero.

Both:

    - Return a pointer on success.
    - Return NULL on allocation failure.
    - Use device-managed cleanup.
    - Usually receive &pdev->dev in a platform driver's probe().
    - Commonly use GFP_KERNEL when allocation may sleep in a
      suitable process context.

Example:

    struct my_device *data;

    data = devm_kzalloc(&pdev->dev, sizeof(*data), GFP_KERNEL);
    if (!data)
        return -ENOMEM;

Use devm_kzalloc() when zero-initialized storage is wanted.
Use devm_kmalloc() when zero initialization is not required and
all fields/bytes will be initialized before they are read.

Do not describe uninitialized memory as having a predictable
"junk" value. Its contents are unspecified and must not be relied on.

Important:

    Zero-initialized memory does not replace the required
    initialization functions for kernel objects such as mutexes,
    spinlocks, work items, timers, and other subsystem objects.


------------------------------------------------------------

106. devm_kcalloc() VS devm_kmalloc_array()
------------------------------------------

Both APIs allocate one contiguous block for an array. They do
not allocate each element as a separate memory block.

Prototypes:

    void *devm_kcalloc(struct device *dev,
                       size_t n,
                       size_t size,
                       gfp_t flags);

    void *devm_kmalloc_array(struct device *dev,
                             size_t n,
                             size_t size,
                             gfp_t flags);

Parameters:

    dev
        Device associated with the managed allocation.

    n
        Number of elements.

    size
        Size in bytes of each element.

    flags
        Allocation flags, commonly GFP_KERNEL when sleeping is
        permitted.

Differences:

    devm_kcalloc()
        Zero-initializes the allocated array and checks for
        overflow when calculating n * size.

    devm_kmalloc_array()
        Does not guarantee zero initialization and checks for
        overflow when calculating n * size.

Example — 64 zero-initialized sensor entries:

    struct sensor_data {
        int temperature;
        int humidity;
    };

    struct sensor_data *data;

    data = devm_kcalloc(&pdev->dev, 64, sizeof(*data), GFP_KERNEL);
    if (!data)
        return -ENOMEM;

    /* All allocated bytes start at zero. */

Example — array without zero initialization:

    struct descriptor *desc;

    desc = devm_kmalloc_array(&pdev->dev, 256,
                              sizeof(*desc), GFP_KERNEL);
    if (!desc)
        return -ENOMEM;

    /* Initialize every descriptor before relying on its fields. */

Why use array helpers?

    They accept the element count and element size separately and
    protect against overflow in the allocation-size multiplication.
    With a manual expression such as count * sizeof(*ptr), overflow
    could otherwise produce an incorrectly small size.

Important:

    Zeroed memory does not automatically make a hardware or DMA
    descriptor valid. Follow the device's descriptor and DMA rules.


------------------------------------------------------------

107. QUICK CHOICE: KERNEL ALLOCATION APIS
----------------------------------------

    kmalloc()
        One allocation; not zero-initialized; caller frees with kfree().

    kzalloc()
        One zero-initialized allocation; caller frees with kfree().

    kmalloc_array()
        Array allocation; not zero-initialized; caller frees with kfree().

    kcalloc()
        Zero-initialized array allocation; caller frees with kfree().

    devm_kmalloc()
        Managed allocation; not zero-initialized.

    devm_kzalloc()
        Managed allocation; zero-initialized.

    devm_kmalloc_array()
        Managed array allocation; not zero-initialized.

    devm_kcalloc()
        Managed, zero-initialized array allocation.

The devm_* allocations are normally cleaned up automatically as
part of the device-managed resource lifecycle. Ordinary allocations
require explicit cleanup on all applicable paths.

Practical guideline:

    One structure that should start at zero:
        devm_kzalloc()

    Array that should start at zero:
        devm_kcalloc()

    Array that does not need zero initialization:
        devm_kmalloc_array()

These are guidelines, not absolute restrictions. Choose the API
based on allocation shape, initialization needs, and lifetime.


------------------------------------------------------------

108. devm_kfree() — EARLY RELEASE OF MANAGED MEMORY
--------------------------------------------------

Prototype:

    void devm_kfree(struct device *dev, const void *p);

Purpose:

    Release a compatible device-managed allocation before its normal
    managed cleanup point.

Example:

    buffer = devm_kzalloc(&pdev->dev, 4096, GFP_KERNEL);
    if (!buffer)
        return -ENOMEM;

    /* Use buffer. */

    devm_kfree(&pdev->dev, buffer);
    buffer = NULL;

The NULL assignment prevents accidental reuse through this variable,
but it does not invalidate any other copies of the pointer.

Matching cleanup APIs:

    kmalloc()/kzalloc()/kmalloc_array()/kcalloc()
        -> kfree()

    devm_kmalloc()/devm_kzalloc()/devm_kmalloc_array()/devm_kcalloc()
        -> no explicit free is normally needed; use devm_kfree() for
           early release when required.

Do not call kfree() on memory allocated by a devm_* allocator.
That bypasses managed-resource bookkeeping and can lead to a later
invalid cleanup or double-free.

Do not use devm_kfree() to free an ordinary kmalloc() allocation.
The allocation and release APIs must match.


------------------------------------------------------------

109. WHAT HAPPENS WHEN probe() FAILS?
------------------------------------

Example:

    static int my_probe(struct platform_device *pdev)
    {
        struct my_device *data;

        data = devm_kzalloc(&pdev->dev, sizeof(*data), GFP_KERNEL);
        if (!data)
            return -ENOMEM;

        /* Initialization later fails. */
        return -EINVAL;
    }

If probe() fails, the driver core cleans up device-managed resources
acquired during that failed probe. The driver does not need to call
kfree() or devm_kfree() for the devm_kzalloc() allocation.

After probe() succeeds, managed resources remain available while the
device is bound and are cleaned up as part of the relevant managed
resource lifecycle, such as driver unbind/removal.

Summary:

    Probe failure:
        Managed resources acquired for that failed probe are cleaned up.

    Probe success followed by device unbind/removal:
        Managed resources are cleaned up during teardown.

    Explicit early release:
        Use the matching managed cleanup API, such as devm_kfree().

Important:

    Automatic memory cleanup does not stop workqueues, kernel threads,
    timers, or other asynchronous activity for you in every case.
    Stop/synchronize execution contexts before freeing data they may use.


------------------------------------------------------------

110. devm_add_action_or_reset()
-------------------------------

Prototype:

    int devm_add_action_or_reset(struct device *dev,
                                 void (*action)(void *),
                                 void *data);

Purpose:

    Register a custom cleanup callback in the device-managed resource
    lifecycle. This is useful for cleanup operations that do not have
    a directly suitable devm_* API.

Parameters:

    dev
        Device that owns the managed cleanup action.

    action
        Callback to execute during cleanup. It receives data as its
        single argument.

    data
        Argument passed to the cleanup callback.

Return:

    0
        The cleanup action was registered successfully.

    Negative error
        Registration failed and action(data) is executed immediately.

The immediate callback on failure is important when a resource has
already been enabled or acquired and must be undone if cleanup cannot
be registered.

Example — register a clock-disable cleanup action:

    static void disable_clock(void *data)
    {
        struct clk *clk = data;

        clk_disable_unprepare(clk);
    }

    ret = devm_add_action_or_reset(&pdev->dev, disable_clock, clk);
    if (ret)
        return ret;

If registration fails, the API already invokes disable_clock(clk).
Do not call the callback a second time in that failure path.
If registration succeeds, the callback runs during managed cleanup.

Cleanup ordering:

    Managed resources/actions are generally released in reverse
    acquisition/registration order. This is useful when a later
    resource depends on an earlier one.

Clock example sequence:

    1. Acquire/enable clock.
    2. Register its cleanup action.
    3. A later initialization step fails.
    4. Failed-probe cleanup runs the registered action.
    5. Clock is disabled/unprepared.

The cleanup callback must be safe both when executed immediately
because registration failed and when executed later during teardown.


------------------------------------------------------------

111. INTERVIEW QUICK REVISION — MANAGED MEMORY
---------------------------------------------

Q1. What is the difference between devm_kmalloc() and devm_kzalloc()?

Answer:

    devm_kmalloc() does not guarantee zero initialization.
    devm_kzalloc() initializes the allocated bytes to zero.

Q2. Do devm_kcalloc() and devm_kmalloc_array() allocate separate
    blocks for each element?

Answer:

    No. Each allocates one contiguous block for the array.

Q3. Why use devm_kcalloc() for an array?

Answer:

    It zero-initializes the array and checks for overflow in the
    element-count times element-size calculation.

Q4. Why must each descriptor be initialized before use after
    devm_kmalloc_array()?

Answer:

    Its memory is not guaranteed to be zero-initialized, so fields
    have unspecified initial contents.

Q5. Can kfree() be used on memory from devm_kzalloc()?

Answer:

    No. Use managed cleanup; devm_kfree() is available for early
    release. Otherwise allow normal managed cleanup.

Q6. What happens to devm_kzalloc() memory if probe() fails?

Answer:

    Managed resources acquired during the failed probe are cleaned up.

Q7. What does devm_add_action_or_reset() do if registration fails?

Answer:

    It executes the supplied cleanup action immediately and returns
    a negative error.

Q8. What does it do if registration succeeds?

Answer:

    It registers the action for execution during managed cleanup.

Q9. Does devm_kzalloc() initialize mutexes and work items completely?

Answer:

    No. Use the appropriate initialization APIs for kernel objects.


------------------------------------------------------------

112. UPDATED TOPICS COMPLETED
-----------------------------

Managed resources and allocation:

    [x] Managed vs non-managed resource lifetime
    [x] Cleanup after probe failure
    [x] Cleanup after successful probe and later unbind/removal
    [x] devm_kzalloc() purpose, prototype, arguments, return
    [x] kzalloc() vs devm_kzalloc()
    [x] kmalloc() vs kzalloc() initialization difference
    [x] devm_kmalloc() vs devm_kzalloc()
    [x] devm_kcalloc()
    [x] devm_kmalloc_array()
    [x] Array allocation and integer-overflow protection
    [x] Why every array entry must be initialized before use
    [x] devm_kfree() and matching allocation/free APIs
    [x] devm_add_action_or_reset()
    [x] Immediate cleanup if action registration fails
    [x] Cleanup actions after failed probe
    [x] General reverse-order managed cleanup concept


------------------------------------------------------------

113. NEXT TOPICS
----------------

Continue in this order:

    1. devm_clk_get() — acquiring a clock handle

    2. Clock acquisition vs clock enable/prepare

    3. Clock cleanup and error handling

    4. Reset controller APIs

    5. GPIO APIs

    6. IRQ retrieval from Device Tree

    7. Device Tree resource parsing and error handling

    8. Platform-driver hands-on implementation

    9. Device Tree overlay / custom DTS

    10. I2C

    11. SPI

Important learning rule:

    Understand what a normal API does before relying on its devm_*
    counterpart. Managed cleanup does not eliminate the need to
    understand resource ownership, ordering, or active execution contexts.


------------------------------------------------------------

END OF CURRENT PLATFORM DRIVER NOTES
------------------------------------


------------------------------------------------------------

CHAT CONTENT — devm_add_action_or_reset()
----------------------------------------

### . Why do we need this API?

Consider a platform driver that enables a hardware clock.
```
Probe()
  |
  ├── Allocate memory
  |
  ├── Enable hardware clock
  |
  └── Initialization complete
```

When the device is removed, the allocated memory can be released automatically. However, **enabling a clock is an operation, not a memory allocation**. We need a way to ensure that the clock is disabled during cleanup.

Linux provides `devm_add_action_or_reset()` for this purpose.

### 2. API prototype
```
int devm_add_action_or_reset(struct device *dev,
                             void (*action)(void *),
                             void *data);
```

Let's understand each parameter.

| Parameter | Meaning                                                         |
| --------- | --------------------------------------------------------------- |
| `dev`     | Device whose managed-resource lifecycle owns the cleanup action |
| `action`  | Function to execute during cleanup                              |
| `data`    | Argument passed to the cleanup function                         |

**Return value:**

- `0`: The cleanup action was registered successfully.
- Negative error: Registration failed, and the API immediately executes the supplied cleanup action.

The second behavior is particularly important.

### 3. Understand the callback

Suppose we have a cleanup function:
```
static void disable_clock(void *data)
{
    struct clk *clk = data;

    clk_disable_unprepare(clk);
}
```

The function receives a pointer to the clock and disables and unprepares it when the registered cleanup action executes.

We can register this function with:
```
ret = devm_add_action_or_reset(&pdev->dev,
                               disable_clock,
                               clk);
```

If registration succeeds, `disable_clock()` runs during managed-resource cleanup.

If registration fails, `disable_clock()` runs immediately.

**Why execute it immediately on failure?**

Because the resource may already have been enabled. If the kernel cannot register the cleanup action, the API invokes the callback immediately to prevent the resource from being left enabled.

### 4. Important: Understand the ordering

Suppose you acquire and enable a clock, then register a cleanup action.
```
1. Acquire clock
2. Enable clock
3. Register cleanup action
4. Continue initialization
```

If registration succeeds, the cleanup action is executed during teardown.

If registration fails, the action runs immediately, so the clock is disabled.

This is the meaning of **"or reset"** in the API name: if registration fails, execute the action immediately to undo the operation.

The callback must be safe to execute immediately on the failure path.

### 5. Interview question

Suppose your platform driver enables a hardware clock successfully.

Then you call:
```
ret = devm_add_action_or_reset(&pdev->dev,
                               disable_clock,
                               clk);
```

The function returns a negative error.

**Question:** Will `disable_clock()` execute immediately, or will it wait until the device is removed?

Explain why.


### 6. Questions and Answers — `devm_add_action_or_reset()`

**Q1. The driver enables a hardware clock and then calls `devm_add_action_or_reset()`. The function returns a negative error. Will `disable_clock()` execute immediately or wait until device removal?**

**Answer:**

`disable_clock()` executes immediately. The cleanup action could not be registered, so the API calls the supplied action immediately to undo the operation. The caller must still handle the returned error.

Example:

```c
ret = devm_add_action_or_reset(&pdev->dev, disable_clock, clk);
if (ret)
    return ret;
```

Do not call `disable_clock()` again in this error path because the API has already called it.


**Q2. The cleanup action is registered successfully. Later, another initialization step fails and `probe()` returns an error. Will the cleanup action run automatically?**

**Answer:**

Yes. When `probe()` fails, the driver core cleans up device-managed resources acquired during that failed probe. The registered cleanup action is executed as part of that cleanup, so the driver does not need to call the action manually on that path.


**Q3. What is the usual cleanup order for device-managed resources?**

**Answer:**

Device-managed resources are generally released in reverse order of acquisition (last acquired, first released). This helps ensure that resources acquired later are cleaned up before earlier resources on which they may depend.

For example, if memory is allocated first, a clock is enabled next, and a cleanup action for the clock is registered afterward, the clock cleanup action normally runs before the earlier memory allocation is released.


**Q4. Why is `devm_add_action_or_reset()` useful for an operation such as enabling a clock?**

**Answer:**

Enabling a clock is an operation, not a memory allocation. `devm_add_action_or_reset()` lets the driver register a callback that reverses the operation during managed-resource cleanup. If callback registration fails, the API runs the callback immediately so the operation is undone rather than leaving the clock enabled without a registered cleanup action.


**Q5. What do the arguments mean in this API?**

```c
int devm_add_action_or_reset(struct device *dev,
                             void (*action)(void *),
                             void *data);
```

**Answer:**

- `dev`: The device whose managed-resource lifecycle owns the cleanup action.
- `action`: The cleanup callback to execute.
- `data`: The pointer passed to the callback when it executes.

The return value is `0` if registration succeeds, or a negative error if registration fails. On registration failure, the callback is executed immediately.


**Q6. Does registering a cleanup action mean the action runs immediately?**

**Answer:**

No. If registration succeeds, the action is retained and runs during managed-resource cleanup, such as when a failed probe is unwound or the device is later detached. It runs immediately only if registration fails.


**Q7. What should the driver do after `devm_add_action_or_reset()` returns an error?**

**Answer:**

Return or otherwise handle the error. Do not manually execute the same cleanup action again, because the API has already executed it on the registration-failure path.


**Q8. What is the key interview takeaway?**

**Answer:**

`devm_add_action_or_reset()` registers a custom cleanup callback tied to a device's managed-resource lifecycle. If registration fails, it immediately executes the callback. If registration succeeds, the callback runs during managed-resource cleanup, generally in reverse acquisition order.


------------------------------------------------------------


============================================================
114. CLOCK MANAGEMENT: devm_clk_get() AND devm_clk_get_enabled()
============================================================

### Why do we need devm_clk_get()?

Many SoC peripherals require a clock to operate.

Examples:

    UART controller
    Ethernet MAC
    I2C controller
    SPI controller
    DMA controller

A peripheral might have registers mapped in memory, but it may not function until its clock is enabled.

Linux provides the Common Clock Framework (CCF) to manage clocks.

The devm_clk_get() API obtains a clock handle associated with a device.

Important distinction:

    Obtaining a clock handle and enabling the clock
    are two different operations.


API prototype:

    struct clk *devm_clk_get(struct device *dev,
                             const char *id);

Parameters:

    dev
        Device requesting the clock, usually &pdev->dev.

    id
        Clock name used to identify the requested clock.

Return value:

    Success: A valid struct clk * handle.

    Failure: An error-encoded pointer.

Unlike devm_kzalloc(), which returns NULL on allocation failure,
devm_clk_get() uses error pointers. Use IS_ERR() and PTR_ERR()
to handle errors.

Example:

    struct clk *clk;

    clk = devm_clk_get(&pdev->dev, "bus");

    if (IS_ERR(clk))
        return PTR_ERR(clk);


### What does the clock name mean?

Example Device Tree node:

    ethernet@40000000 {
        compatible = "example,my-ethernet";
        reg = <0x40000000 0x1000>;

        clocks = <&clock_controller 5>;
        clock-names = "bus";
    };

The clock-names property identifies the clock using the name "bus".
The driver can request it using:

    clk = devm_clk_get(&pdev->dev, "bus");

The actual clock name and clock specification depend on the hardware,
Device Tree bindings, and clock provider.


### Does devm_clk_get() enable the clock?

No. devm_clk_get() obtains the clock handle. It does not enable the clock.

To enable the clock, Linux provides:

    int clk_prepare_enable(struct clk *clk);

Example:

    clk = devm_clk_get(&pdev->dev, "bus");

    if (IS_ERR(clk))
        return PTR_ERR(clk);

    ret = clk_prepare_enable(clk);

    if (ret)
        return ret;

After clk_prepare_enable() succeeds, the clock is prepared and enabled.
The driver must later balance this operation with:

    clk_disable_unprepare(clk);

Managed acquisition of a clock handle does not automatically balance
a successful clk_prepare_enable() call.


### devm_clk_get() vs devm_clk_get_enabled()

    devm_clk_get()
        Obtains a managed clock handle. It does not enable the clock.

    devm_clk_get_enabled()
        Obtains the clock handle and prepares/enables the clock.
        Managed cleanup handles disabling/unpreparing the clock and
        releasing the handle at the appropriate lifecycle point.

Conceptual prototype:

    struct clk *devm_clk_get_enabled(struct device *dev,
                                     const char *id);

Example:

    struct clk *clk;

    clk = devm_clk_get_enabled(&pdev->dev, "bus");

    if (IS_ERR(clk))
        return PTR_ERR(clk);

After this call succeeds:

    - A valid clock handle is available.
    - The clock is prepared and enabled.
    - Managed cleanup handles disabling/unpreparing the clock and
      releasing the handle.

Availability of this API depends on the Linux kernel version.


### API comparison

    devm_clk_get()
        Obtains clock handle: Yes
        Enables clock: No
        Releases handle automatically: Yes
        Automatically disables clock: No, not merely from obtaining handle

    devm_clk_get_enabled()
        Obtains clock handle: Yes
        Enables clock: Yes
        Releases handle automatically: Yes
        Automatically disables clock: Yes, through managed cleanup


### Clock enabled manually: cleanup responsibility

Consider:

    clk = devm_clk_get(&pdev->dev, "bus");
    if (IS_ERR(clk))
        return PTR_ERR(clk);

    ret = clk_prepare_enable(clk);
    if (ret)
        return ret;

    /* Hardware initialization fails */
    return -EINVAL;

This code is incomplete. After clk_prepare_enable() succeeds, the clock
remains enabled. Returning -EINVAL does not automatically disable it.

The driver must arrange cleanup on every relevant error path. One option
is to register a cleanup action immediately after enabling the clock:

    ret = clk_prepare_enable(clk);
    if (ret)
        return ret;

    ret = devm_add_action_or_reset(&pdev->dev, disable_clock, clk);
    if (ret)
        return ret;

Here disable_clock() calls clk_disable_unprepare(). If registration
succeeds, the action executes during managed-resource cleanup. If
registration fails, devm_add_action_or_reset() invokes the action
immediately.

Do not disable the same clock a second time after registration failure,
because the API has already executed the cleanup action.


### Interview questions and answers — clock management

Q1. If devm_clk_get() succeeds, can we conclude that the clock is enabled
and the peripheral is ready?

Answer:

No. devm_clk_get() obtains the clock handle but does not enable the clock.
The driver must enable it separately if required, using
clk_prepare_enable(). Even an enabled clock does not by itself guarantee
that the peripheral is fully initialized or ready for operation.


Q2. What is wrong with returning an error after clk_prepare_enable()
succeeds?

Answer:

If a later hardware initialization step fails, simply returning an error
does not disable the clock. The driver must arrange to disable and
unprepare the clock on that error path. A managed cleanup action can be
registered using devm_add_action_or_reset().


Q3. If a cleanup action for the enabled clock is successfully registered
and probe() later fails, what happens?

Answer:

The driver core cleans up device-managed resources acquired during the
failed probe. The registered clock cleanup action executes automatically.


Q4. If probe() succeeds and the device is later unbound, will a
successfully registered clock cleanup action execute automatically?

Answer:

Yes. The cleanup action is tied to the device-managed resource lifecycle
and executes during managed-resource cleanup. Managed resources are
generally released in reverse order of acquisition, which helps preserve
dependency ordering.


Q5. What is the difference between devm_clk_get() and
 devm_clk_get_enabled()?

Answer:

devm_clk_get() obtains a managed clock handle but does not enable the
clock. devm_clk_get_enabled() obtains the handle and prepares/enables the
clock, and registers managed cleanup for the enabled clock.


Q6. If devm_clk_get_enabled() succeeds and a later initialization step
fails, must the driver manually call clk_disable_unprepare() before
returning?

Answer:

No. devm_clk_get_enabled() registers managed cleanup for the enabled
clock. If probe() subsequently fails, the driver core cleans up the
managed resources, including disabling and unpreparing the clock.


Q7. What is the key interview point about managed clock acquisition?

Answer:

Managed acquisition of a clock handle does not automatically balance a
separate successful clk_prepare_enable() call. Use devm_clk_get_enabled()
when appropriate, or arrange explicit/managed cleanup after enabling a
clock manually.


### Question to answer next

Suppose probe() succeeds using devm_clk_get_enabled(). Later, the device
is unbound from the driver.

Will the clock be automatically disabled, or must the driver's remove()
callback disable it manually? Why?


------------------------------------------------------------
END OF CLOCK MANAGEMENT NOTES SO FAR
------------------------------------------------------------
 

============================================================
114. CHAT TRANSCRIPT — CLOCK MANAGEMENT
============================================================

Assistant:

## Next Topic: `devm_clk_get()` — Linux Platform Drivers

We'll continue with clock management, which is an important topic for Linux platform driver development and semiconductor interviews.

### 1. Why do we need `devm_clk_get()`?

Many SoC peripherals require a clock to operate.

Examples:
- UART controller
- Ethernet MAC
- I2C controller
- SPI controller
- DMA controller

A peripheral might have registers mapped in memory, but it may not function until its clock is enabled.

For example, imagine an Ethernet MAC peripheral.

```text
          Linux Platform Driver
                  |
                  v
           Obtain clock handle
                  |
                  v
            Enable the clock
                  |
                  v
           Initialize Ethernet MAC
                  |
                  v
           Start Ethernet operation
```

Linux provides the **Common Clock Framework (CCF)** to manage clocks.

The `devm_clk_get()` API obtains a clock handle associated with a device.

**Important distinction:** Obtaining a clock handle and enabling the clock are two different operations.

---

### 2. API prototype

```c
struct clk *devm_clk_get(struct device *dev,
                         const char *id);
```

Let's understand each parameter.

| Parameter | Meaning |
|---|---|
| `dev` | Device requesting the clock, usually `&pdev->dev` |
| `id` | Clock name used to identify the requested clock |

**Return value:**

- Success: A valid `struct clk *` handle.
- Failure: An error-encoded pointer.

Unlike `devm_kzalloc()`, which returns `NULL` on allocation failure, `devm_clk_get()` uses **error pointers**.

Therefore, use `IS_ERR()` and `PTR_ERR()` to handle errors.

Example:

```c
struct clk *clk;

clk = devm_clk_get(&pdev->dev, "bus");

if (IS_ERR(clk))
    return PTR_ERR(clk);
```

Here:
- `IS_ERR(clk)` checks whether the returned pointer represents an error.
- `PTR_ERR(clk)` extracts the negative error code.

---

### 3. What does the clock name mean?

Consider a Device Tree node:

```dts
ethernet@40000000 {
    compatible = "example,my-ethernet";
    reg = <0x40000000 0x1000>;

    clocks = <&clock_controller 5>;
    clock-names = "bus";
};
```

The `clock-names` property identifies the clock using the name `"bus"`.

The driver can request it using:

```c
clk = devm_clk_get(&pdev->dev, "bus");
```

Conceptually:

```text
Device Tree
    |
    | clock-names = "bus"
    v
devm_clk_get(&pdev->dev, "bus")
    |
    v
Clock handle
    |
    v
struct clk *
```

**Important:** The actual clock name and clock specification depend on the hardware, Device Tree bindings, and clock provider.

---

### 4. Does `devm_clk_get()` enable the clock?

**No. This is a very important interview point.**

`devm_clk_get()` obtains the clock handle. It does not enable the clock.

To enable the clock, Linux provides:

```c
int clk_prepare_enable(struct clk *clk);
```

Example:

```c
struct clk *clk;
int ret;

/* Step 1: Obtain the clock handle */
clk = devm_clk_get(&pdev->dev, "bus");
if (IS_ERR(clk))
    return PTR_ERR(clk);

/* Step 2: Prepare and enable the clock */
ret = clk_prepare_enable(clk);
if (ret)
    return ret;

/* Clock is now enabled */

/* Continue device initialization */
```

**Important:** This example is incomplete regarding cleanup. If a later initialization step fails, the driver must disable the clock before returning. We'll learn how to handle that safely using `devm_add_action_or_reset()`.

### 5. Interview question

Consider this code:

```c
clk = devm_clk_get(&pdev->dev, "bus");

if (IS_ERR(clk))
    return PTR_ERR(clk);
```

Suppose this code executes successfully.

**Question:** Can you conclude that the hardware clock is enabled and the peripheral is ready to operate?

Try answering in your own words first. The answer is below as covered in the chat.

User:

clock handle is optained,clock not enabled. to enable clock prepare clock enable.

Assistant:

**Correct! ✅** You understand the difference between obtaining a clock handle and enabling the clock.

### 1. Your answer — interview-ready version

"`devm_clk_get()` obtains a clock handle associated with the device, but it does not enable the clock. To enable it, I use `clk_prepare_enable()`. If that operation succeeds, I must eventually balance it with `clk_disable_unprepare()`."

### 2. Understand the complete sequence

```c
struct clk *clk;
int ret;

/* Step 1: Obtain the clock handle */
clk = devm_clk_get(&pdev->dev, "bus");
if (IS_ERR(clk))
    return PTR_ERR(clk);

/* Step 2: Prepare and enable the clock */
ret = clk_prepare_enable(clk);
if (ret)
    return ret;

/* Clock is now enabled */

/* Continue device initialization */
```

**Important:** This example is incomplete regarding cleanup. If a later initialization step fails, the driver must disable the clock before returning. We'll learn how to handle that safely using `devm_add_action_or_reset()`.

### 3. Remember these three APIs

| API | Purpose | Cleanup |
|---|---|---|
| `devm_clk_get()` | Obtain a clock handle | Handle released automatically |
| `clk_prepare_enable()` | Prepare and enable the clock | Must be balanced |
| `clk_disable_unprepare()` | Disable and unprepare the clock | Called when the clock is no longer needed |

**Key interview point:** Managed acquisition of a clock handle does not automatically balance a successful `clk_prepare_enable()` call.

---

### 4. Next question

Consider this code:

```c
clk = devm_clk_get(&pdev->dev, "bus");
if (IS_ERR(clk))
    return PTR_ERR(clk);

ret = clk_prepare_enable(clk);
if (ret)
    return ret;

/* Hardware initialization fails */
return -EINVAL;
```

**Question:** What is wrong with this code when hardware initialization fails? What should the driver do before returning `-EINVAL`?

Try answering in your own words.

User:

before returning clock must be released. kernel do resource clean up for device associated resource. custom clean up function can be registered.

Assistant:

Good! ✅ You correctly identified that the clock needs cleanup and that we can register a custom cleanup function. Let's clarify one important detail.

### 1. What is wrong with the code?

```c
ret = clk_prepare_enable(clk);
if (ret)
    return ret;

/* Hardware initialization fails */
return -EINVAL;
```

After `clk_prepare_enable()` succeeds, the clock remains enabled. If hardware initialization fails, simply returning `-EINVAL` does **not** automatically disable the clock.

Why? Because `devm_clk_get()` manages the clock handle's lifetime, but it does not automatically balance `clk_prepare_enable()`.

### 2. Option A: Manual cleanup

You can disable the clock before returning the error:

```c
ret = initialize_hardware();
if (ret) {
    clk_disable_unprepare(clk);
    return ret;
}
```

However, you must handle every relevant error path correctly.

### 3. Option B: Register automatic cleanup

You can use `devm_add_action_or_reset()`.

After successfully enabling the clock:

```c
ret = clk_prepare_enable(clk);
if (ret)
    return ret;

ret = devm_add_action_or_reset(&pdev->dev,
                               disable_clock,
                               clk);
if (ret)
    return ret;
```

Here, `disable_clock()` calls `clk_disable_unprepare()`. If registration succeeds, the action executes during managed-resource cleanup. If registration fails, the API executes the action immediately.

**Important:** The cleanup action must be registered immediately after enabling the clock, and the code must not disable the same clock a second time.

---

### 4. Next interview question

Suppose you use `devm_add_action_or_reset()` to register the clock cleanup action.

Later, `probe()` succeeds. Eventually, the device is removed.

**Question:** Will the clock cleanup action execute automatically? At what point in the device removal process should it happen relative to other managed resources?

Try answering in your own words.

User:

custom clean up funtion registered in device managed resource. when device is deteched. cleanup automatically triggered. there resource are released opposite to allocated order. so the dependacies handled properly

Assistant:

**Excellent! ✅ Your answer is correct.** You understand both automatic cleanup and the reverse order of resource release.

### 1. Review your answer

You said:
> Custom cleanup function registered in device-managed resource. When device is detached, cleanup automatically triggered. Resources are released in the opposite order to allocation, so dependencies are handled properly.

That's right!

Let's refine one detail: managed resources are generally released in **reverse order of acquisition**. This helps ensure that resources acquired later are cleaned up before resources they depend on.

### 2. Example: Clock and memory cleanup

Suppose your driver performs these operations:

```c
/* 1. Allocate device data */
data = devm_kzalloc(&pdev->dev, sizeof(*data), GFP_KERNEL);
if (!data)
    return -ENOMEM;

/* 2. Obtain clock handle */
clk = devm_clk_get(&pdev->dev, "bus");
if (IS_ERR(clk))
    return PTR_ERR(clk);

/* 3. Enable clock */
ret = clk_prepare_enable(clk);
if (ret)
    return ret;

/* 4. Register clock cleanup */
ret = devm_add_action_or_reset(&pdev->dev,
                               disable_clock, clk);
if (ret)
    return ret;
```

Assuming the clock cleanup action is successfully registered and no other resources are acquired afterward, cleanup proceeds conceptually as follows:

```text
Device unbind/removal
        |
        v
Disable and unprepare clock
        |
        v
Release clock handle
        |
        v
Free device data
```

The clock is disabled before the earlier-acquired resources are released.

**Important:** This example illustrates managed-resource ordering. Real drivers must also account for interrupts, workqueues, DMA activity, and other operations that might still access the device.

### 3. Interview-ready answer

"`devm_add_action_or_reset()` registers a custom cleanup callback with the device-managed resource system. If registration succeeds, the action executes during managed-resource cleanup when the device is unbound or the relevant lifecycle ends. Resources are generally released in reverse acquisition order, which helps maintain correct dependency ordering."

---

## Next topic: Clock management — `devm_clk_get()`

You've now learned the purpose of `devm_clk_get()`, `clk_prepare_enable()`, `clk_disable_unprepare()`, and `devm_add_action_or_reset()`.

Let's go deeper into clock acquisition.

**Question:** What is the difference between these two APIs?

```c
devm_clk_get(&pdev->dev, "bus");
```

```c
devm_clk_get_enabled(&pdev->dev, "bus");
```

Think about what each API does to the clock and which cleanup responsibilities it handles.

User:

clk get gets clock handle . it pointer to clock configuration sturceture.
devm clk get enable acutally enables the clock

Assistant:

**Correct! ✅** You understood the main difference. Let's refine your answer for a Linux driver interview.

## 1. `devm_clk_get()` vs `devm_clk_get_enabled()`

### A. `devm_clk_get()`

**Purpose:** Obtains a clock handle associated with the device. It does not enable the clock.

Conceptual prototype:

```c
struct clk *devm_clk_get(struct device *dev, const char *id);
```

Example:

```c
struct clk *clk;

clk = devm_clk_get(&pdev->dev, "bus");
if (IS_ERR(clk))
    return PTR_ERR(clk);
```

After this call succeeds:
- You have a valid clock handle.
- The clock is not necessarily enabled.
- You must enable it separately if your driver requires it.
- The clock handle is released automatically according to the device-managed resource lifecycle.

### B. `devm_clk_get_enabled()`

**Purpose:** Obtains the clock handle and enables the clock.

Conceptual prototype:

```c
struct clk *devm_clk_get_enabled(struct device *dev,
                                 const char *id);
```

Example:

```c
struct clk *clk;

clk = devm_clk_get_enabled(&pdev->dev, "bus");
if (IS_ERR(clk))
    return PTR_ERR(clk);
```

After this call succeeds:
- You have a valid clock handle.
- The clock is prepared and enabled.
- The managed cleanup mechanism handles disabling and unpreparing the clock and releasing the handle at the appropriate lifecycle point.

This avoids writing separate clock-enabling and cleanup code in many drivers.

**Note:** Availability of this API depends on the Linux kernel version.

## 2. Compare the APIs

| Feature | `devm_clk_get()` | `devm_clk_get_enabled()` |
|---|---|---|
| Obtains clock handle | Yes | Yes |
| Enables clock | No | Yes |
| Automatically releases handle | Yes | Yes |
| Automatically disables clock | No, not merely from acquiring the handle | Yes, through its managed cleanup |
| Requires manual enabling | If needed | No |

### Interview-ready answer

"`devm_clk_get()` obtains a managed clock handle but does not enable the clock. `devm_clk_get_enabled()` obtains the handle and prepares and enables the clock. It also registers the appropriate managed cleanup, simplifying error handling and device removal."

---

## 3. Next question

Consider this code:

```c
clk = devm_clk_get_enabled(&pdev->dev, "bus");
if (IS_ERR(clk))
    return PTR_ERR(clk);

/* Hardware initialization fails */
return -EINVAL;
```

**Question:** If hardware initialization fails, do you need to call `clk_disable_unprepare(clk)` manually before returning `-EINVAL`?

Explain why.

User:

kernal core do clean up device associated resource. it doesn't perform clock disable, other periphereal disable. so u should call clk diable unprepare .

Assistant:

Not quite. You're applying the correct general concept of resource cleanup, but **`devm_clk_get_enabled()` is different from `devm_clk_get()`**.

### 1. What happens when `probe()` fails?

Your code:

```c
clk = devm_clk_get_enabled(&pdev->dev, "bus");
if (IS_ERR(clk))
    return PTR_ERR(clk);

/* Hardware initialization fails */
return -EINVAL;
```

If `devm_clk_get_enabled()` succeeds, it:
1. Obtains the clock handle.
2. Prepares and enables the clock.
3. Registers managed cleanup for the enabled clock.

When `probe()` subsequently fails, the driver core cleans up the resources acquired during that failed probe. The managed clock cleanup disables and unprepares the clock and releases the handle.

**Therefore, you do not need to call `clk_disable_unprepare()` manually in this example.**

### 2. Compare the two approaches

| API | If a later initialization step fails |
|---|---|
| `devm_clk_get()` followed by `clk_prepare_enable()` | You must arrange to disable and unprepare the clock. |
| `devm_clk_get_enabled()` | Managed cleanup handles disabling and unpreparing the clock. |

This is one of the advantages of using the managed API.

### 3. Interview-ready answer

"`devm_clk_get_enabled()` obtains the clock handle, prepares and enables the clock, and registers managed cleanup. If `probe()` subsequently fails, the driver core cleans up the managed resources, including disabling and unpreparing the clock. Therefore, I don't need to call `clk_disable_unprepare()` manually."

---

### 4. Next question

Suppose `probe()` succeeds using `devm_clk_get_enabled()`.

Later, the device is unbound from the driver.

**Will the clock be automatically disabled, or must the driver's `remove()` callback disable it manually? Why?**

User:

append it in readme.



============================================================
115. CLOCK MANAGEMENT: RATE, PARENT, ACCURACY AND STATE APIs
============================================================

This section records the required concepts from the clock-management lessons. It is a concise reference, not a verbatim chat transcript.

### 115.1 clk_set_rate()

Prototype:

    int clk_set_rate(struct clk *clk, unsigned long rate);

Purpose:

    Requests a change to the clock rate. The requested rate is expressed in Hz.

Return value:

    0       The rate-change request succeeded according to the clock framework.
    < 0     The request failed; the value is a negative error code.

Important points:

    - A successful return does not independently measure or prove the physical clock frequency.
    - The achievable rate depends on the clock provider, parent clock, dividers, hardware constraints, and shared consumers.
    - Use clk_get_rate() to query the rate reported by the clock framework after the request.
    - Request the required rate before starting peripheral operations when possible.
    - Changing a rate while enabled may be supported, but only do so when the clock provider and peripheral hardware permit it safely.

Example:

    ret = clk_set_rate(clk, 100000000UL); /* request 100 MHz */
    if (ret)
        return ret;

    rate = clk_get_rate(clk);

### 115.2 clk_round_rate()

Prototype:

    long clk_round_rate(struct clk *clk, unsigned long rate);

Purpose:

    Determines a rate the clock provider can provide for a requested rate, without itself changing the clock rate.

Example:

    rounded_rate = clk_round_rate(clk, 75000000UL);
    if (rounded_rate < 0)
        return rounded_rate;

    if (rounded_rate != 75000000L)
        return -EINVAL; /* only if the peripheral requires exactly 75 MHz */

    ret = clk_set_rate(clk, 75000000UL);
    if (ret)
        return ret;

Important points:

    - If a 75 MHz request rounds to 80 MHz, the clock has not changed merely because clk_round_rate() returned 80 MHz.
    - Accept the rounded rate only if it falls within the peripheral's permitted operating range.
    - A rounded result is not a guarantee that a later rate change will succeed if clock constraints or configuration change.

### 115.3 clk_get_parent()

Prototype:

    struct clk *clk_get_parent(struct clk *clk);

Purpose:

    Returns a handle to the clock's current parent. It does not enable the parent or change the clock configuration.

Return value:

    A parent clock handle, or NULL if there is no parent.

Important point:

    Obtaining a handle to the parent does not prove that the parent clock is enabled.

### 115.4 clk_set_parent()

Prototype:

    int clk_set_parent(struct clk *clk, struct clk *parent);

Purpose:

    Requests that a clock use a different parent, if supported by the clock provider and hardware.

Return value:

    0       Success.
    < 0     Failure.

Important points:

    - Changing the parent may change the output frequency.
    - The provider may adjust a divider to preserve the previous rate if supported, but the driver must not assume this will happen.
    - Check the resulting framework-reported rate with clk_get_rate().
    - Consider peripheral limits and other consumers of shared clocks.

### 115.5 clk_get_accuracy()

Prototype:

    unsigned long clk_get_accuracy(struct clk *clk);

Purpose:

    Returns the accuracy information reported by the clock framework/provider, expressed in parts per billion (ppb). This is reported information, not an independent runtime measurement.

Distinction:

    clk_get_rate()       Reports the clock rate in Hz.
    clk_get_accuracy()   Reports clock accuracy information in ppb.

Conversion:

    accuracy_ppm = accuracy_ppb / 1000

For example, 50000 ppb corresponds to 50 ppm. Accuracy is not the clock frequency; a 100 MHz clock with 50 ppm accuracy has a nominal frequency of 100 MHz, with a tolerance described by that accuracy specification.

Note: Check the target kernel's documentation for the exact semantics and handling of unknown accuracy values.

### 115.6 clk_is_enabled()

Prototype:

    int clk_is_enabled(struct clk *clk);

Purpose:

    Checks the enabled state reported by the clock framework/provider.

Typical return values:

    1       Enabled.
    0       Disabled.
    < 0     Error, where supported/reported by the implementation.

Important points:

    - Obtaining a clock handle with devm_clk_get() does not enable the clock.
    - A return value of 1 does not prove that the current driver enabled the clock. Another consumer may have enabled it.
    - Use this API for state inspection, not as a replacement for balanced clock enable/disable management.
    - Exact behavior can depend on the clock provider and target kernel implementation.

### 115.7 Safe clock configuration sequence

A common sequence when the hardware permits it is:

    1. Obtain the clock handle.
    2. Request the required clock rate.
    3. Check the result and, if needed, query the reported rate.
    4. Enable the clock.
    5. Initialize and operate the peripheral.

If the peripheral requires a particular reset, idle, or clock-gating sequence, follow its datasheet. Do not assume that every clock rate or parent change requires disabling the clock, or that every change is safe while the peripheral is active.

### 115.8 Clock management interview questions and answers

Q1. Can a clock's frequency be changed while it is enabled?

Answer: Sometimes. It depends on the clock provider and hardware, and on whether the peripheral can safely tolerate the change. Configure the required rate before starting peripheral operations when possible.

Q2. Does clk_set_rate() returning 0 prove the physical frequency is exactly the requested value?

Answer: No. It means the rate-change request succeeded according to the clock framework. Query clk_get_rate() for the framework-reported rate; the API does not independently measure the physical signal.

Q3. Does clk_round_rate() change the clock frequency?

Answer: No. It determines a rate the provider can offer for the request. clk_set_rate() is used to request the change.

Q4. If clk_round_rate() returns 80 MHz for a 75 MHz request, should the driver accept it?

Answer: Only if 80 MHz is within the peripheral's allowed operating range. If exactly 75 MHz is required, reject the rounded rate.

Q5. Does clk_get_parent() enable the parent clock?

Answer: No. It returns a handle to the current parent clock.

Q6. Must the output frequency change after clk_set_parent() succeeds?

Answer: Not necessarily. The provider may adjust a divider or other configuration to preserve the rate, if supported. Verify the reported rate with clk_get_rate().

Q7. If clk_is_enabled() returns 1, does that prove this driver enabled the clock?

Answer: No. The clock may have been enabled by another consumer. The API reports enabled state, not which consumer enabled it.

Q8. If clk_prepare_enable() fails, should the driver call clk_disable_unprepare() for that failed attempt?

Answer: No. Balance a successful enable operation with clk_disable_unprepare(); do not perform a matching disable for an enable attempt that failed.

Q9. What is the difference between clock rate and clock accuracy?

Answer: clk_get_rate() reports the rate in Hz. clk_get_accuracy() reports accuracy information in ppb. Accuracy is not a frequency value.

Q10. Why configure the clock before starting peripheral operations when possible?

Answer: It allows the peripheral to start at its required rate and avoids potentially disruptive rate changes during active transfers. The hardware specification determines the safe sequence.

Q11. If clk_set_parent() succeeds, will the divider always be adjusted automatically to preserve the old rate?

Answer: No. This depends on the provider and hardware; the driver must not assume automatic rate preservation.

Q12. Does obtaining a clock handle mean that the clock is enabled?

Answer: No. Getting a handle and enabling a clock are separate operations.


END OF CLOCK MANAGEMENT API REFERENCE


====================================================================
GPIO CONSUMER DRIVER — DEEP-DIVE NOTES AND TRICK POINTS
====================================================================

1. GPIO CONSUMER VS GPIO CONTROLLER
-----------------------------------

GPIO consumer driver:

    Uses a GPIO to control or monitor a device, for example:
        - LED enable
        - Reset signal
        - Button/status input
        - Peripheral interrupt signal

    Common API family:
        devm_gpiod_get()
        gpiod_get_value()
        gpiod_set_value()
        gpiod_to_irq()

GPIO controller driver:

    Implements the GPIO controller itself, commonly using gpiochip APIs.
    This is a different task from writing a peripheral driver that consumes GPIOs.

TRICK POINT:

    For peripheral-driver development, learn the GPIO consumer API first.
    Do not confuse gpiod_* consumer APIs with gpiochip_* controller APIs.


2. WHAT IS A GPIO DESCRIPTOR?
-----------------------------

    struct gpio_desc represents a GPIO connection requested through the
    Linux GPIO subsystem. A driver usually holds a pointer to this descriptor.

Example:

    struct gpio_desc *reset_gpio;

    reset_gpio = devm_gpiod_get(&pdev->dev, "reset", GPIOD_OUT_LOW);

Important distinctions:

    Physical GPIO pin:
        The actual signal connection on the board/controller.

    struct gpio_desc:
        Kernel-side object representing the requested GPIO connection.

    reset_gpio:
        The driver's pointer to that descriptor.

TRICK POINT:

    A GPIO descriptor is not a physical pin number and not an MMIO register
    address. Use the descriptor with GPIO consumer APIs rather than directly
    manipulating GPIO-controller registers from a consumer driver.


3. ACQUIRING A GPIO — devm_gpiod_get()
--------------------------------------

Purpose:

    Request a GPIO connection and obtain a descriptor for later operations.

Why use it:

    It lets the driver refer to a named connection such as reset rather than
    hard-coding a board-specific GPIO number. Firmware can map that connection
    to the correct line on each supported board.

When to use it:

    In probe(), when the GPIO is required for the device to operate. Use the
    optional variant only when the hardware can genuinely function without it.

Prototype:

    struct gpio_desc *devm_gpiod_get(struct device *dev,
                                     const char *con_id,
                                     enum gpiod_flags flags);

Arguments:

    dev:
        Consumer device, typically &pdev->dev inside platform-driver probe().

    con_id:
        Connection identifier. For a Device Tree property named reset-gpios,
        the connection identifier is "reset" (without the -gpios suffix).

    flags:
        Initial direction/value request, such as GPIOD_IN, GPIOD_OUT_LOW,
        or GPIOD_OUT_HIGH.

Return handling:

    Valid pointer:
        GPIO descriptor acquired successfully.

    ERR_PTR(...):
        Failure. Check with IS_ERR() and obtain the error with PTR_ERR().

Example:

    reset_gpio = devm_gpiod_get(&pdev->dev, "reset", GPIOD_OUT_LOW);
    if (IS_ERR(reset_gpio))
        return dev_err_probe(&pdev->dev, PTR_ERR(reset_gpio),
                             "Failed to get reset GPIO\n");

TRICK POINTS:

    - Declare a pointer first does not acquire a GPIO; the acquisition API does.
    - Check IS_ERR() before using the returned pointer.
    - -EPROBE_DEFER can mean a GPIO provider/dependency is not ready; propagate
      the error so the driver core can retry.
    - GPIOD_OUT_LOW/HIGH set the initial LOGICAL value, not necessarily the
      physical voltage. Active-low polarity affects the physical level.


4. OPTIONAL GPIO — devm_gpiod_get_optional()
--------------------------------------------

Purpose:

    Acquire a named GPIO when it exists while allowing a genuinely optional
    connection to be absent.

Why use it:

    It distinguishes a valid absent optional connection (NULL) from a real
    acquisition error (ERR_PTR), so the driver can handle both correctly.

When to use it:

    Only when the hardware design and driver behavior support operation without
    that GPIO. Do not use it to hide a missing required reset or power-control line.

Prototype:

    struct gpio_desc *devm_gpiod_get_optional(struct device *dev,
                                              const char *con_id,
                                              enum gpiod_flags flags);

Results:

    Valid pointer:
        GPIO exists and was acquired.

    NULL:
        GPIO is legitimately absent.

    ERR_PTR(...):
        Actual acquisition failure; check IS_ERR() and use PTR_ERR().

TRICK POINT:

    Check IS_ERR() before checking for NULL. An error pointer is non-NULL.
    Use the optional variant only if the hardware can operate without that GPIO;
    do not make a required reset/power GPIO optional just to hide probe failures.


5. GPIO DIRECTION AND VALUE ARE DIFFERENT OPERATIONS
----------------------------------------------------

Purpose:

    Direction APIs configure whether the line is input or output; value APIs
    read or set the logical signal state.

Why keep them separate:

    A value operation does not configure direction. Separating the operations
    makes initialization and later direction changes explicit.

When to use them:

    Prefer acquisition flags such as GPIOD_IN or GPIOD_OUT_LOW/HIGH for initial
    setup. Call direction APIs later only when a direction change is actually needed.

Common APIs:

    int gpiod_direction_input(struct gpio_desc *desc);
    int gpiod_direction_output(struct gpio_desc *desc, int value);
    int gpiod_get_direction(const struct gpio_desc *desc);
    int gpiod_get_value(const struct gpio_desc *desc);
    void gpiod_set_value(struct gpio_desc *desc, int value);

Meaning:

    gpiod_direction_input():
        Configure as input; returns 0 or a negative error.

    gpiod_direction_output():
        Configure as output and set the initial logical value; returns 0 or
        a negative error.

    gpiod_get_direction():
        Common return convention is 0 for output, 1 for input, negative error
        if direction cannot be determined. Check the target kernel/controller.

    gpiod_get_value():
        Read logical value (0 or 1), or a negative error where supported.

    gpiod_set_value():
        Set logical output value; return type is void, so no error is returned
        directly by this accessor.

TRICK POINTS:

    - Direction is not the same as value. Setting a value does not configure
      direction.
    - If acquisition used GPIOD_IN or GPIOD_OUT_LOW/HIGH, direction and initial
      value were requested during acquisition; do not redundantly configure the
      same direction unless you need to change it.
    - gpiod_get_direction() is not required just to remember the direction your
      own driver requested; use it only when checking direction is meaningful.
    - Do not interpret every nonzero return from an int-returning GPIO API as
      an error: for gpiod_get_value(), 1 is a valid logical HIGH. Check value < 0.


6. LOGICAL VALUE VS PHYSICAL VOLTAGE — ACTIVE-LOW TRICK
------------------------------------------------------

Example Device Tree property:

    reset-gpios = <&gpio 22 GPIO_ACTIVE_LOW>;

For a GPIO described as active-low:

    Logical value 1:
        Signal is active; physical level is typically LOW.
        For reset, this normally means reset asserted.

    Logical value 0:
        Signal is inactive; physical level is typically HIGH.
        For reset, this normally means reset deasserted.

TRICK POINT:

    Do not assume logical 1 always means physical HIGH. GPIO descriptor accessors
    account for active-low polarity when using logical accessors. Use raw accessors
    only when the driver genuinely needs physical-level semantics and understands
    the implications.


7. SLEEPABLE GPIO ACCESSORS
---------------------------

Purpose:

    The *_cansleep() variants support GPIO providers whose read/write operation
    may need to sleep, for example a GPIO expander accessed over I2C or SPI.

Why use them:

    A GPIO access may involve a bus transaction. Using the correct accessor and
    execution context avoids sleeping in atomic or hard-interrupt context.

When to use them:

    Use *_cansleep() from a sleepable context whenever the provider may sleep.
    Use the non-cansleep variant only when the provider operation is atomic-safe.
    gpiod_cansleep(desc) can help determine the provider's constraint.

Common pairs:

    gpiod_get_value()             gpiod_get_value_cansleep()
    gpiod_set_value()             gpiod_set_value_cansleep()

Additional helper:

    bool gpiod_cansleep(const struct gpio_desc *desc);

    Purpose: report whether accessing this descriptor may sleep.
    Argument: desc is the already-acquired GPIO descriptor.
    Return: true if access may sleep; false otherwise.
    Use: choose a safe accessor/context. This does not make a sleepable
    operation legal in hard-IRQ context; move it to a threaded handler or
    deferred work when appropriate.

Some GPIO controllers, including GPIO expanders reached over I2C/SPI, may need
sleepable bus transactions to read or change a GPIO.

Rules:

    - Use the *_cansleep() accessor when the GPIO controller may sleep.
    - Do not call a sleepable accessor from hard-IRQ/atomic context.
    - If an IRQ thread must access an I2C/SPI GPIO expander, a threaded handler
      is generally appropriate; ordinary hard-IRQ context is not.

TRICK POINT:

    The *_cansleep suffix is about whether the GPIO operation may sleep, not
    whether the consumer device itself is an I2C/SPI device. The controller
    providing the GPIO determines this behavior.


8. OPTIONAL GPIO ACQUISITION AND RESOURCE LIFETIME
--------------------------------------------------

    gpiod_get():
        Non-managed acquisition; release with gpiod_put().

    devm_gpiod_get():
        Device-managed acquisition; release is handled by the devres lifecycle.

    devm_gpiod_get_optional():
        Same managed lifetime, but absence may be represented by NULL.

TRICK POINTS:

    - Do not call gpiod_put() for a descriptor obtained with devm_gpiod_get().
    - Managed cleanup releases the descriptor; it does not guarantee the physical
      pin is driven to a particular safe level. Explicitly put hardware into the
      required safe state before detach if the device requires it.
    - Stop asynchronous activity and ensure handlers/work cannot use driver data
      after its lifetime ends.


9. GPIO TO IRQ — gpiod_to_irq()
-------------------------------

Purpose:

    Translate a GPIO descriptor into the Linux IRQ number associated with that
    line, when the GPIO provider supports interrupt mapping.

Why use it:

    GPIO line numbers and Linux IRQ numbers belong to different namespaces; the
    driver must not guess an IRQ number from a GPIO pin number.

When to use it:

    After acquiring/configuring the GPIO input and before registering an IRQ handler.

Prototype:

    int gpiod_to_irq(const struct gpio_desc *desc);

Purpose:

    Maps a GPIO descriptor to an IRQ number when the GPIO/controller supports
    GPIO-to-IRQ mapping. Returns an IRQ number on success or a negative error.

TRICK POINTS:

    - This does not register an interrupt handler.
    - Check irq < 0 before using the result.
    - A GPIO is not guaranteed to support interrupts simply because it can be
      used as a GPIO.


10. THREADED IRQ — PRIMARY HANDLER VS THREAD FUNCTION
----------------------------------------------------

Purpose:

    Register an interrupt handler, optionally splitting fast hard-IRQ work from
    processing that must run in a sleepable IRQ thread.

Why use it:

    The primary handler cannot sleep, but device processing may need a mutex or
    I2C/SPI transactions. The threaded handler provides the appropriate context.

When to use it:

    Use a threaded handler when interrupt processing needs sleepable operations.
    Keep any primary handler short and non-sleeping; use a threaded-only setup
    when its API requirements and hardware behavior fit the design.

Prototype:

    int devm_request_threaded_irq(struct device *dev,
                                 unsigned int irq,
                                 irq_handler_t handler,
                                 irq_handler_t thread_fn,
                                 unsigned long irqflags,
                                 const char *devname,
                                 void *dev_id);

Primary handler:

    Runs in hard-IRQ context. Must not sleep. It may quickly identify/acknowledge
    an interrupt if hardware allows that access in this context and return
    IRQ_WAKE_THREAD to request threaded processing.

Thread function:

    Runs in IRQ thread/process context and may perform sleepable operations such
    as I2C transactions or mutex acquisition, subject to normal kernel rules.

Threaded-only configuration:

    A NULL primary handler is allowed for a threaded IRQ. The IRQ core supplies
    a default primary handler that wakes the thread. For this standard setup,
    use IRQF_ONESHOT as required by the IRQ API contract.

TRICK POINTS:

    - The kernel's default primary handler only wakes the IRQ thread; it does not
      implement the device-specific interrupt processing for the driver.
    - Do not perform I2C/SPI transactions, take a mutex, or call other sleepable
      functions from a hard-IRQ handler.
    - IRQF_ONESHOT keeps the IRQ masked while the threaded handler runs; it does
      not debounce a button and does not guarantee every physical transition is
      separately remembered.
    - IRQF_TRIGGER_RISING requests a rising edge; IRQF_TRIGGER_FALLING requests
      a falling edge; OR them together to request both edges where supported.
    - For shared IRQs, handler/flags and a unique non-NULL dev_id must satisfy the
      shared-IRQ requirements. A threaded-only handler is not a drop-in solution
      for every shared-line design.
    - devm_request_threaded_irq() is device-managed. Do not also free that same
      registration manually with free_irq().


11. GPIO DEBOUNCING
-------------------

Purpose:

    Request a debounce interval to suppress rapid transitions such as mechanical
    switch contact bounce, if the GPIO provider supports debounce.

Why use it:

    One button press can generate several electrical edges, which otherwise may
    be interpreted as multiple events.

When to use it:

    During setup when the signal needs debounce. Check the return value; do not
    assume unsupported hardware debounce automatically falls back to software debounce.

Prototype:

    int gpiod_set_debounce(struct gpio_desc *desc,
                           unsigned int debounce);

The debounce interval is specified in microseconds. For example, 20000 requests
20 ms, if supported by the GPIO controller.

Purpose:

    Mechanical switches can produce several rapid transitions during one physical
    press/release. Debouncing avoids treating the bounce as multiple events.

TRICK POINTS:

    - gpiod_set_debounce() requests controller/hardware debounce; it does not
      automatically implement a software timer-based debounce fallback.
    - Support depends on the GPIO controller. Check and handle a negative return.
    - If hardware debounce is unsupported, use a suitable software strategy,
      such as delayed work/timer-based rechecking, designed around the device.


12. GPIO DEVICE TREE CONNECTION NAMING
--------------------------------------

Example:

    my_device {
        compatible = "vendor,my-device";
        reset-gpios = <&gpio 22 GPIO_ACTIVE_LOW>;
    };

Consumer call:

    devm_gpiod_get(&pdev->dev, "reset", GPIOD_OUT_LOW);

Mapping:

    reset-gpios  -> connection ID "reset"
    button-gpios -> connection ID "button"
    enable-gpios -> connection ID "enable"

TRICK POINTS:

    - Drop the -gpios suffix from con_id.
    - The GPIO specifier's number/cells and flags are controller/binding-specific;
      don't copy illustrative pin numbers into a real board DTS without checking
      that controller's binding and the board schematic.
    - Ensure the GPIO property belongs to the device node that the driver is
      actually using to acquire the GPIO.


13. GPIO DEBUGGING CHECKLIST
----------------------------

If acquisition fails:

    1. Check the exact probe error with dmesg/journalctl -k.
    2. Confirm the expected Device Tree node and property exist.
    3. Check connection-ID spelling: "reset" for reset-gpios, not "reset-gpios".
    4. Check whether the error is -EPROBE_DEFER and identify the missing provider.
    5. Check pinctrl/mux configuration, GPIO ownership/conflicts, polarity and
       board wiring as applicable.

If the value is wrong:

    1. Check active-low polarity and whether the API returns logical or raw value.
    2. Check direction and whether the pin is multiplexed for another function.
    3. Check external pull-ups/pull-downs and actual voltage with suitable tools.
    4. Use an oscilloscope/logic analyzer when software logs cannot prove the
       electrical level or edge timing.

If an IRQ does not fire:

    1. Check gpiod_to_irq() return value.
    2. Check trigger edge and physical polarity.
    3. Check controller/board support for GPIO interrupts.
    4. Check IRQ registration return value and relevant kernel logs.
    5. Confirm the pin actually transitions electrically.
    6. Check IRQ state/line sharing and handler return behavior as applicable.

If an IRQ fires repeatedly for one button press:

    1. Consider switch bounce.
    2. Check edge selection and electrical noise/pull configuration.
    3. Check hardware debounce support or implement a suitable software strategy.
    4. Do not assume IRQF_ONESHOT is a debounce mechanism.

Useful inspection tools include dmesg/journalctl -k, relevant /sys/kernel/debug
entries when enabled, dynamic debug, IRQ information under /proc/interrupts,
and an oscilloscope/logic analyzer for electrical evidence. Availability depends
on kernel configuration and platform.


14. GPIO INTERVIEW TRICK POINTS — QUICK REFERENCE
------------------------------------------------

Q: Is struct gpio_desc the physical GPIO number?
A: No. It is a kernel-side descriptor representing the GPIO connection.

Q: Does devm_gpiod_get() configure direction?
A: It requests the initial direction/value according to the flags provided.

Q: Is logical 1 always physical HIGH?
A: No. GPIO_ACTIVE_LOW can invert the physical level represented by logical values.

Q: Why check value < 0 rather than if (value)?
A: For gpiod_get_value(), 1 is a valid logical HIGH and 0 is a valid LOW; negative
   values indicate errors where the accessor reports them.

Q: Can gpiod_get_value() be used from a hard-IRQ handler for every GPIO?
A: No. Some GPIO controllers can sleep. Use *_cansleep() only from sleepable
   context when the controller requires it.

Q: Does gpiod_to_irq() register the handler?
A: No. It only maps the descriptor to an IRQ number when supported.

Q: Can a threaded IRQ handler perform an I2C transfer?
A: Yes, if the operation is otherwise valid; it runs in a sleepable thread context.

Q: What does IRQF_ONESHOT do?
A: Keeps the IRQ masked while the threaded handler runs. It does not debounce.

Q: Does devm_gpiod_get() guarantee a safe physical level during driver removal?
A: No. Managed cleanup releases the resource; safe hardware-state requirements
   must be handled explicitly by the driver where necessary.

Q: What is the difference between GPIO consumer and GPIO controller drivers?
A: The consumer uses GPIOs; the controller driver implements the GPIO provider.


15. GPIO LEARNING STATUS
------------------------

Theory covered in this learning sequence:

    - GPIO consumer vs controller
    - GPIO descriptor concept
    - devm_gpiod_get() and devm_gpiod_get_optional()
    - Direction and value APIs
    - Logical value vs active-low physical level
    - Sleepable GPIO accessors
    - gpiod_to_irq()
    - Primary handler vs threaded handler
    - IRQF_ONESHOT and edge-trigger flags
    - gpiod_set_debounce()
    - Device Tree connection IDs
    - Error handling and initial debugging checklist

Still required for practical completion:

    - Build and load a GPIO consumer driver in the target environment.
    - Verify Device Tree connection and driver probe.
    - Test input/output and active-low behavior on actual hardware.
    - Register and test a GPIO interrupt.
    - Reproduce and debug an intentional failure.
    - Verify cleanup/unbind behavior and document the evidence.

NOTE:

    The notes above are a learning reference. Confirm exact API behavior and
    signatures against the target kernel version and the specific GPIO controller.




====================================================================
