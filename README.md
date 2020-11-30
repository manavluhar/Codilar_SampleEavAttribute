## Product EAV Attribute

 - [ ] A data patch is a class that contains data modification instructions. it implements DataPatchInterface.
 - [ ] Through patch versioning, all created attribute can be replaced.
 - **apply()**  function is used to define your core logic for installing/upgrading data to the database.

- **getAliases()**  function defines aliases for the patch class. In this case, the class name could possibly change, and if it does, we should supply the old class-name here, so that it doesn’t get executed a second time.

- **getDependencies()**  function contain the class name of dependent patches. This functionality notifies Magento to execute the “patches”. We first define it here, before the patch script.

- **getVersion()**  function can return a version number of the patch. If the version number of the module is higher than the version we specify in our patch, then it will not get executed. If it is equal to or lower than the version here, it will be executed.
- **revert()** function - when a module is uninstalled the logic in the revert() method is called.

***creating data patch***
```
<?php  
namespace Codilar\SampleEavAttribute\Setup\Patch\Data;  
  
use Magento\Eav\Setup\EavSetupFactory;  
use Magento\Framework\Setup\ModuleDataSetupInterface;  
use Magento\Framework\Setup\Patch\DataPatchInterface;  
use Magento\Framework\Setup\Patch\PatchVersionInterface;  
  
class ProductAttribute implements DataPatchInterface, PatchVersionInterface  
{  
	/**  
	 * @var ModuleDataSetupInterface   
	 */  
	private $moduleDataSetup;  
	/**  
	 * @var EavSetupFactory   
	 */  
	private $eavSetupFactory;  
	  
	public function __construct(  
	    ModuleDataSetupInterface $moduleDataSetup,  
	    EavSetupFactory $eavSetupFactory  
	) {  
	    $this->moduleDataSetup = $moduleDataSetup;  
	    $this->eavSetupFactory = $eavSetupFactory;  
	}
}
```

Defining **apply()** function

```
public function apply()  
{  
    $eavSetup = $this->eavSetupFactory->create(['setup' => $this->moduleDataSetup]);  
    $eavSetup->addAttribute(  
        \Magento\Catalog\Model\Product::ENTITY,  
        'sample_attribute',  
        [  
            'type' => 'text',  
            'backend' => '',  
            'frontend' => '',  
            'label' => 'Sample Atrribute',  
            'input' => 'text',  
            'class' => '',  
            'source' => '',  
            'global' => \Magento\Eav\Model\Entity\Attribute\ScopedAttributeInterface::SCOPE_GLOBAL,  
            'visible' => true,  
            'required' => true,  
            'user_defined' => false,  
            'default' => '',  
            'searchable' => false,  
            'filterable' => false,  
            'comparable' => false,  
            'visible_on_front' => false,  
            'used_in_product_listing' => true,  
            'unique' => false,  
            'apply_to' => ''  
  ]  
    );  
}
```
Defining **getDependencies()** function

```
public static function getDependencies()  
{  
    return [];  
}
```
Defining **getVersion()** function

```
public static function getVersion()  
{  
    return '1.0.0';  
}
```
Defining **getAliases()** function

```
public function getAliases()  
{  
    return [];  
}
```
## Customer EAV Attribute
 - [ ]  if we want to enable rollback for the patch during module uninstallation, then **PatchRevertableInterface** must implemented
 
 ***creating data patch***
```
<?php  
  
namespace Codilar\SampleEavAttribute\Setup\Patch\Data;  
  
use Magento\Customer\Setup\CustomerSetup;  
use Magento\Customer\Setup\CustomerSetupFactory;  
use Magento\Framework\Setup\ModuleDataSetupInterface;  
use Magento\Framework\Setup\Patch\DataPatchInterface;  
use Magento\Framework\Setup\Patch\PatchRevertableInterface;  
use Magento\Framework\Setup\Patch\PatchVersionInterface;  
  
class CustomerAttribute implements DataPatchInterface, PatchRevertableInterface, PatchVersionInterface  
{  
    /**  
	 * @var ModuleDataSetupInterface  
	 */  
  private $moduleDataSetup;  
    /**  
	 * @var CustomerSetup  
	 */  
  private $customerSetupFactory;  
  
    /**  
	 * Constructor * * @param ModuleDataSetupInterface $moduleDataSetup  
	 * @param CustomerSetupFactory $customerSetupFactory  
	 */  
  public function __construct(  
        ModuleDataSetupInterface $moduleDataSetup,  
        CustomerSetupFactory $customerSetupFactory  
  ) {  
        $this->moduleDataSetup = $moduleDataSetup;  
        $this->customerSetupFactory = $customerSetupFactory;  
    }  
}
```

Defining **apply()** function

```
public function apply()  
{  
    $this->moduleDataSetup->getConnection()->startSetup();  
    
    /** @var CustomerSetup $customerSetup */  
	$customerSetup = $this->customerSetupFactory->create(['setup' => $this->moduleDataSetup]);  
    $customerSetup->addAttribute(  
        \Magento\Customer\Model\Customer::ENTITY,  
        'customer_pan_number',  
        [  
            'type' => 'varchar',  
            'label' => 'Customer Pan Number',  
            'input' => 'text',  
            'source' => '',  
            'required' => false,  
            'visible' => true,  
            'position' => 500,  
            'system' => false,  
            'backend' => ''  
	    ]  
    );  
  
    $attribute = $customerSetup->getEavConfig()->getAttribute('customer', 'customer_pan_number')->addData([  
        'used_in_forms' => [  
            'adminhtml_customer'  
		  ]  
    ]);  
    $attribute->save();  
    
    $this->moduleDataSetup->getConnection()->endSetup();  
}
```
Defining **revert()** function
```
public function revert()  
{  
    $this->moduleDataSetup->getConnection()->startSetup();  
    /** @var CustomerSetup $customerSetup */  
    $customerSetup = $this->customerSetupFactory->create(['setup' => $this->moduleDataSetup]);  
    $customerSetup->removeAttribute(\Magento\Customer\Model\Customer::ENTITY, 'customer_pan_number');  
  
    $this->moduleDataSetup->getConnection()->endSetup();  
}
```
Defining **getDependencies()** function

```
public static function getDependencies()  
{  
    return [];  
}
```
Defining **getVersion()** function

```
public static function getVersion()  
{  
    return '1.0.0';  
}
```
Defining **getAliases()** function

```
public function getAliases()  
{  
    return [];  
}
```
>Notes:<br/>  
> - customer_pan_number is a custom customer attribute<br/>   
> - passed used_in_forms array value adminhtml_customer, that means it is available in admin customer edit form.<br/>
> you can pass as per requirement<br>**as a example:**
'used_in_forms' => [
'adminhtml_checkout',
'adminhtml_customer',
'adminhtml_customer_address',
'customer_account_edit',
'customer_address_edit',
'customer_register_address',
'customer_account_create']
  
  Run the following commands
```
php bin/magento setup:upgrade
php bin/magento cache:flush
```

Run the following command to revert all `composer` installed data patches:
```
bin/magento module:uninstall Codilar_SampleEavAttribute
```
