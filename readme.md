This project is an example of using the maintainable xml converter from Kuali Rice 2.x to do "just-in-time" conversion on maintenance document XML. It also includes patches to existing maintenance XML conversion code from Rice 2.0 so that the conversion rules file includes more conversions, as well as allowing for a custom rules file to be supplied. An application wanting to use a custom file would simply copy MaintainableXMLUpgradeRules.xml, add any additional rules, and then reference that as appropriate in Rice xml configuration (see information on the default configuration below).

This project just includes a few of the files that can be "patched" to accomplish this. A summary is below:

* MaintenanceDocumentBase - includes a modification to getDataObjectFromXML which attempts to convert the maintenance document XML if it fails to reconstitute a business object from the given maintainable XML.
* MaintainableXMLConversionService - a newly introduced interface which simply allows us to fetch the conversion service from spring via a service locator
* KRADServiceLocatorWeb - modified version of this service locator that adds a getter for the MaintainableXMLConversionService
* MaintainableXMLConversionServiceImpl - a modified implementation of the maintainable xml conversion service which allows for a custom rule file to be supplied
* MaintainableXMLUpgradeRules.xml - a modified list of conversion rules, the original list supplied in Rice 2 code has a number of gaps which we encountered


Also, the original IU code where this modification was made also set up the following default configuration in a Rice xml config file:

```XML
<param name="maintainable.conversion.rule.file" override="false">/org/kuali/rice/krad/config/MaintainableXMLUpgradeRules.xml</param>
```

In order to apply these changes, you would patch your copy of Rice code with the changes.

Note that all of the changes here are based on Rice 2.2.3 source code! So if you are using a different version you will probably want to hand-pick the changes and apply them to the version of Rice you are using.
