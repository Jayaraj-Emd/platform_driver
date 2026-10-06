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
