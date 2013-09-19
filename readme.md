Rice XML Converter
==================

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

Maintainable Document XML Conversion
====================================

In Rice 2.0 some properties on maintainable objects have been changed/removed which can cause issues opening older maintenance documents since those documents contain invalid XML. This converter handles this situation and transforms the XML which is stored in the maintenance document table before XStream parses it for display on the page to help resolve this problem. If your maintainable classes have changed during an upgrade to Kuali Rice 2.x you may need to add some additional configuration to get your maintenance documents to work properly. This consists of two parts:

1. Create a file with the conversion rules for how to convert the maintainable document XML (see below for an example).
2. Modify the "maintainable.conversion.rule.file" configuration parameter to point to your XML file with the conversion rules.

Note: If your maintainables haven't changed during the upgrade it is likely that you will not need to do any additional configuration for your older maintenance documents to work properly.

Maintainable XML conversion rule file
-------------------------------------

This file contains two types of mappings:

1. Maintainable implementation class in 1.x to the implementation class in 2.x
2. A mapping of a 2.x maintainable implementation class to a map of property names which have changed between the 1.x and 2.x versions.

For example, say you had the following class which you use for a maintainable object with Rice 1.0.3:

```Java
public class FooImpl {
  public String property1;
  public String property2;
}
```

Which has changed to the following in your new version of the code with Rice 2.x:

```Java
public class FooBo {
  public String newProperty1;
  public String property2;
}
```

The maintenance document XML converter would use XML like the following to correct the XML from the database:

```XML
<rules>

  <!-- Rules for any changes to the fully qualified class name
        Uses regex to match.
  -->
  <rule name="maint_doc_classname_changes" alsoRenameClass="true">
    <pattern>
      <match>FooImpl</match>
      <replacement>FooBo</replacement>
    </pattern>
  </rule>

  <rule name="maint_doc_changed_class_properties">
    <pattern>
      <class>FooBo</class>
      <pattern>
        <match>property1</match>
        <replacement>newProperty1</replacement>
      </pattern>
    </pattern>
  </rule>
</rules>
```

The conversion file for the Rice maintainable documents is called MaintainableXMLUpgradeRules.xml. You can refer to this XML file for more realistic examples if needed.