# fhir.versions.r3r4 - EXPERIMENTAL
fhir r3r4 maps in an implementation guide and npm package. 

ATTENTION: This is an experimental implementation guide, currently the R3R4 maps are in the standard itself, see [FHIR source github](https://github.com/HL7/fhir/tree/master/implementations/r3maps/R3toR4). ®© HL7.org 2011+. FHIR Release 4 (v4.0.0) 

[The specification](https://www.hl7.org/fhir/r3maps.html) states that the maps are subject to ongoing maintenance using the FHIR NPM Package "fhir.versions.r3r4" which is maintained on [GitHub](https://github.com/FHIR/r3r4). This repository/ig will be submitted as a pull request for HL7, see [Zulip discusion](https://chat.fhir.org/#narrow/stream/179166-implementers/topic/Maps.20np.20mpackage.20for.20R3.20to.20R4.3F).

1. Changes between FHIR source github for r3r4 maps:
* renamed BodyStructure.map to BodySite.map
* renamed CoverageEligibilityRequest.map to EligibilityRequest.map
* renamed CoverageEligibilityResponse.map to EligibilityResponse.map
* renamed MolecularSequence3.map to Sequence.map

2. Changes between FHIR source github for r4r3 maps
* renamed MolecularSequence to Sequence.map

3. StructureMap/AdverseEvent3to4: StructureMap.group[1].rule[5].name	error	id value 'use description in leiu of type for event' is not valid
changed to a line comment in AdverseEvent3to4.map

4. Multiple maps have a group with no rule inside, but [StructureMap](http://www.hl7.org/fhir/structuremap.html) requires at least one rule in a group 

group Age(source src : AgeR3, target tgt : Age) extends Quantity <<type+>> {
}

To be clarified.

5. Contained Map in a transform is not referenced and gives an error in the validation process (e.g. Claim3to4.map):

If the resource is contained in another resource, it SHALL be referred to from elsewhere in the resource or SHALL refer to the containing resource ( (unmatched: Use)) [contained.where(((('#' + id) in (.descendants().reference | .descendants().as(canonical) | .descendants().as(uri) | .descendants().as(url))) or descendants().where(reference = '#').exists() or descendants().where(as(canonical) = '#').exists() or descendants().where(as(canonical) = '#').exists()).not()).trace('unmatched', id).empty()]

map is:

```
map "http://hl7.org/fhir/StructureMap/Claim3to4" = "R3ToR4ConversionsForClaim"

conceptmap "Use" {
  prefix s = "http://hl7.org/fhir/claim-use"
  prefix t = "http://hl7.org/fhir/claim-use"

  s:complete - t:claim
  s:proposed - t:preauthorization
  s:exploratory - t:predetermination
}

uses "http://hl7.org/fhir/3.0/StructureDefinition/Claim" alias ClaimR3 as source
uses "http://hl7.org/fhir/StructureDefinition/Claim" alias Claim as target

imports "http://hl7.org/fhir/StructureMap/*3to4"

group Claim(source src : ClaimR3, target tgt : Claim) extends DomainResource <<type+>> {
  src.identifier -> tgt.identifier;
  src.status -> tgt.status;
  src.type -> tgt.type;
  src.subType -> tgt.subType;
  src.use as v -> tgt.use = translate(v, '#Use', 'code');

  ....

```
  <rule>
      <name value="use"/>
      <source>
        <context value="src"/>
        <element value="use"/>
        <variable value="v"/>
      </source>
      <target>
        <context value="tgt"/>
        <contextType value="variable"/>
        <element value="use"/>
        <transform value="translate"/>
        <parameter>
          <valueId value="v"/>
        </parameter>
        <parameter>
          <valueString value="#Use"/>
        </parameter>
        <parameter>
          <valueString value="code"/>
        </parameter>
      </target>
    </rule>
```
fhirPath rule checks not parameter/valueString #Use element, but pararmeter can also not be valueReference (only id|string|boolean|integer|decimal)
zulip or gfore topic?

6. Issue with div transformation between R3 and R4 and back

div seems to StringType in R4 instead of xhtml from in npm package definition, why?

frst problem that R3 TO R4 transformation did not show div Element, adjusted code but maybe wrongly

second problem ist that

 rule : div; vars = source variables [src: (Narrative)], target variables [tgt: (Narrative)], shared variables []
source variables [src: (Narrative), vvv: "<div xmlns="http://www.w3.org/1999/xhtml">HACC Funded Billing for Peter James Chalmers</div>"], target variables [tgt: (Narrative), vvv: (xhtml)], shared variables []


org.hl7.fhir.exceptions.FHIRException: No matches found for rule for 'string to xhtml' from http://hl7.org/fhir/StructureMap/Narrative4to3, from rule 'div'
    at org.hl7.fhir.r4.utils.StructureMapUtilities.resolveGroupByTypes(StructureMapUtilities.java:1563)
    at org.hl7.fhir.r4.utils.StructureMapUtilities.executeRule(StructureMapUtilities.java:1410)
    at org.hl7.fhir.r4.utils.StructureMapUtilities.executeGroup(StructureMapUtiliti

7. Using R3R4ConversionTests

* Loaded fhir.core.4.0.0 package directly -> 66 errrors (otherwise code did not works anymor)
* Loaded maps from IG -> duplicate problems with versions, code fix necessarc
* still the same errors

/Users/oliveregger/Documents/github/fhir/implementations/r3maps/outcoumes.json