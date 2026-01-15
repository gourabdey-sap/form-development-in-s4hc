INTRODUCTION
In #SAP S/4HANA Cloud Public Edition, there are scenarios where standard output forms and data sources do not meet business requirements. This blog post explains how to create custom form using a custom data source in SAP S/4HC (Grow with SAP system).

The form is developed directly in the S/4HANA Cloud tenant. Please note that this blog post does not cover “#SAP Forms service by Adobe” on #SAP Business Technology Platform. Instead, it focuses on form development within the public cloud system itself (Developer Extensibility)

When Should You Follow This Approach?
You should consider creating a custom form with a custom data source in the following scenarios:

SAP has not provided a standard data source for the required business object.
Examples include:

Purchase Requisition output

Supplier Invoice output (MIRO)

You have developed a custom application or custom business object, and an output form is required.
Since the development is custom, there will naturally be no standard form or data source available for output generation

In such cases, defining a custom data source and binding it to a custom form becomes the correct and often the only viable approach.

When Should You Not Follow This Approach?
You should avoid this approach when a standard solution already exists:

If SAP provides a standard data source and a standard form template, you should use them instead of creating custom ones.

Example: Purchase Order outputs already have standard data sources and form templates available.

When standard data sources are available, always prefer the “Form Template” Fiori app to customize the output.

Customization of Content Form Templates - SAP S/4HANA Finance

Prerequisites 
Tools
Adobe LiveCycle Designer
Required for designing and modifying Adobe-based output forms. The tool is used to define the layout, structure, and binding of the form fields to the data source.
ABAP Development Toolkit (ADT)
Required for designing and modifying Adobe-based output forms. The tool is used to define the layout, structure, and binding of the form fields to the data source.
Skills
Knowledge on Adobe Form
Knowledge on RAP Development Framework
Code References
All example code referenced in this article is available in the Git repository linked below. To keep the blog post concise and focused, the source code is not reproduced here in full, as including it would unnecessarily lengthen the post.

GitHub - gourabdey-sap/form-development-in-s4hc: Custom Form Development with Custom Data Source in S4HC



SO, LET'S START THE DEVELOPMENT ACTIVITIES
As an example for custom form development using a custom data source, we will use the Purchase Requisition (PR) form. This example is chosen because the Purchase Requisition process is well known and familiar to most readers, making it easier to understand and relate to the concepts discussed.

Step 1 - Custom CDS View Creation
We will create a RAP BO for Purchase Requisition.

I usually prefer defining a Root CDS view to keep the design future-proof. This approach allows for easy functional enhancements later without requiring changes to the existing CDS hierarchy.

That said, if your objective is only to call the form, a standard CDS view is sufficient. In such cases, creating a Root CDS view (RAP Business Object) is not mandatory.

We will create two interface CDS views and two corresponding consumption CDS views:

One set for header data

One set for item data



The source code for the CDS can be found in Git Hub repo linked above.

Step 2 - Marking the Header Consumption CDS as Form Data Provider
Add the following annotation to the header consumption CDS view. This annotation is required only for the header consumption view and must not be added to the item CDS view

@ObjectModel.supportedCapabilities: [ #OUTPUT_FORM_DATA_PROVIDER  ]


Step 3 - Creating Form Data Provider (Data Source)
Next, we need to create a Service Definition to act as the form data provider (data source). Both the header and item entities must be exposed in this service definition. 

An important point to note is the leading entity annotation, defined using @ObjectModel.leadingEntity.name

@EndUserText.label: 'PR Form Data Provider'
@ObjectModel.leadingEntity.name: 'ZMM_C_PRHEADER'
define service ZMM_PR_FORM_DATA {
  expose ZMM_C_PrHeader as Header;
  expose ZMM_C_PrItem   as Item;
}
Key Point Consideration:
Only one CDS entity must be designated as the Form Data Provider. In this scenario, the header CDS view (ZMM_C_PrHeader) is the sole data provider and is annotated with @ObjectModel.supportedCapabilities: [ #OUTPUT_FORM_DATA_PROVIDER].
The item CDS view must not contain this annotation.
The Service Definition should have the leading entity defined in upper case. The Data Definition name of the header view is ZMM_C_PrHeader still, the leading entity has to be provided as ZMM_C_PRHEADER.
@ObjectModel.leadingEntity.name: 'ZMM_C_PRHEADER'​
Service Binding creation is not required for rendering the form. Only Service Binding is enough,
But, still in this blog post Service Binding will be required for generation of RAP Application. 
Step 4 - Form Creation
Create new Form using Path: Right Click on Package > New > Other ABAP Repository Object > Form. Provide the form name as required. I have given as ZMM_PR_OUTPUT.



Data Provider: 
Select the data provider as the Service Definition been created in Step 3. In this example, this will be ZMM_PR_FORM_DATA. 



Save the form (Ctrl/Cmd + S). 

Please note: The form cannot be activated yet as the layout is still not developed. Refer to Step

Step 5 - Download XML Schema (XSD File)
To download the XSD file of form, click on "Download Schema" button.



Click on export and give the file name. I have given the file name as "PR Schema.xsd".



Step 6 - Create Form Layout
Open Adobe LiveCycle Designer (ALD) locally in the system. You can search for the application "Adobe LiveCycle Designer" in your system and should be able to open it, provided this is installed locally in your system.

Once the Adobe LiveCycle Designer is open, click on New Form. Or alternatively you can use the menu File > New Form (Ctrl/Cmd + N) 



Select the orientation as required and click on Finish.



Use the menu File > New Data Connection to link to the XSD file.



Select the XSD file downloaded from Step 5. Click on finish.



The field details can be seen the Data View tab. If you don't have data view tab, then you can add it from the menu Windows > Data View



Rename the Page and bind it to the header Node.



Then save the form as XDP file (Ctrl/Cmd + S).



Step 7 - Designing the Form
As both form is created and the connection is established with XSD file, we can work on the layout development as required.

As an example, I will have a very basic layout where some of the fields from header will be bound and we will have a table which will have item details. 



You would need the data in XML format. We will download it subsequent section. We will also see how to preview the form locally in the system. 

Step 8 - Generate Sample Data
Right click on Preview PDF tab > Select Preview Tab

Click on Generate Preview Data > Click on the button below Data File Name

Provide the name of the file and Click Save. 

Then click on Generate Button, which will generate some sample data. Click on Ok.

With this we have some example data been generated and we can Preview the form locally.



Step 9 - Preview the Form
Click on the Preview Tab to have a look at the form layout been developed.



This is how the form layout appears locally during preview. Some fields are displayed in light blue in the local preview; however, when the form is executed within S/4HANA Cloud (S/4HC), this blue coloring will not be visible.

Step 10 - Upload the Form in S4
Open the form been created in Step 4 and upload the developed form using Upload Layout. 



Now you should be able to activate the form.



So, we have successfully created the form and uploaded in S4HC.

Step 11 - Adding Attachment Related field in Header Consumption View
For showing the attachment using CDS, we would need below 3 fields

File Name - Name of the file
Mime Type - For PDF, this will be always
Attachment - The PR Output PDF
So, we will achieve this by using virtual element of CDS. 

Important Annotation:
Virtual Field

@ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_MM_PR_FORM'
Attachment Field

@Semantics.largeObject:{
  mimeType: 'MimeType',
  fileName: 'FileName',
  contentDispositionPreference: #ATTACHMENT
}
Mime Type

@Semantics.mimeType: true
CDS Source Code
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Purchase Requisition Header'
@ObjectModel.supportedCapabilities: [ #OUTPUT_FORM_DATA_PROVIDER ]
@Metadata.allowExtensions: true
define root view entity ZMM_C_PrHeader
  provider contract transactional_query
  as projection on ZMM_I_PrHeader
{
  key     PurchaseRequisition,
          PurReqnDescription,
          PurchaseRequisitionType,
          LastChangeDateTime,
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_MM_PR_FORM'
  virtual FileName   : abap.char( 50 ),
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_MM_PR_FORM'
          @Semantics.mimeType: true
  virtual MimeType   : abap.char( 50 ),
          @ObjectModel.virtualElementCalculatedBy: 'ABAP:ZCL_MM_PR_FORM'
          @Semantics.largeObject:{
            mimeType: 'MimeType',
            fileName: 'FileName',
            contentDispositionPreference: #ATTACHMENT
          }
  virtual Attachment : abap.rawstring( 0 ),
          /* Associations */
          _Item : redirected to composition child ZMM_C_PrItem
}
Step 12 - Logic for Virtual Elements 
The calculation class for virtual element should implement the interface if_sadl_exit_calc_element_read.

The interface has 2 methods.

get_calculation_info :
The fields which are used in the logic required to be added to the field et_requested_orig_elements.
For our case, only Purchase Requisition number is enough.
calculate:
The actual logic for virtual fields will go here. 
The required fields have to be populated in internal table ct_calculated_data.
Step 13 - Calling the Adobe Form
Rending the Form (line no 23-31)
The form is rendered using the method cl_fp_ads_util=>render_pdf.
This requires the data and the layout.
Reading data in XML format (line 9-14)
The XML data are being read using the object of CL_FP_FDP_SERVICES. 
It requires the key of the Root/Header CDS, which is defined as lead entity in the Service Definition in Step 3. In our case, only Purchase Requisition is the key. If multiple keys are there, value should be passed for all keys.
The Purchase requisition data is fetched in the line 13 in the variable LV_DATA in XSTRING format.
Reading data in Form Layout (line no 19)
We would also need the form layout (Adobe Form which was designed and uploaded in Step 10) to render the form.
The form layout is fetched and assigned to LV_XML in XSTRING format.
Getting reference to Form API
Passing the Service Definition created in Step 3 to get the reference.
This API will be used to get the data from the Form Data Provider in XML format.
The parameter iv_max_depth represents navigation level. In this example, we have Header and Item so, 1 level depth should be fine
  METHOD get_pdf.

    mc_duplicate_call = abap_true.

    DATA(lo_fdp_api) = cl_fp_fdp_services=>get_instance(
      iv_max_depth          = 1
      iv_service_definition = `ZMM_PR_FORM_DATA` ).

    DATA(lt_keys)    = lo_fdp_api->get_keys( ).

    lt_keys[ name = 'PURCHASEREQUISITION' ]-value = im_v_pr.

    DATA(lv_data) = lo_fdp_api->read_to_xml_v2(
      it_select = lt_keys
    ).

    DATA(lv_string) = cl_abap_conv_codepage=>create_in( )->convert( lv_data ).

    DATA(lv_xml) = lo_fdp_api->get_xsd_v2( ).

    DATA(lv_string2) = cl_abap_conv_codepage=>create_in( )->convert( lv_xml ).

    DATA(lo_reader) = cl_fp_form_reader=>create_form_reader( `ZMM_PR_OUTPUT` ).

    DATA(ls_layout) = lo_reader->get_layout( ).

    cl_fp_ads_util=>render_pdf( EXPORTING iv_xml_data   = lv_data
                                          iv_xdp_layout = ls_layout
                                          iv_locale     = 'en_US'
                                IMPORTING ev_pdf        = rt_v_pdf
                                         ).

    mc_duplicate_call = abap_false.

  ENDMETHOD.
Step 14 - Service Binding
Create a Service Binding for the RAP application. This should be OData v4, as attachment does not work in OData V2 using CDS Annotation.



Preview the Application. 



