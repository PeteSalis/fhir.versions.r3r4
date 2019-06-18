# fhir.versions.r3r4 - EXPERIMENTAL
fhir r3r4 maps in an implementation guide and npm package. 

ATTENTION: This is an experimental implementation guide, currently the R3R4 maps are in the standard itself, see [FHIR source github](https://github.com/HL7/fhir/tree/master/implementations/r3maps/R3toR4). ®© HL7.org 2011+. FHIR Release 4 (v4.0.0) 

[The specification](https://www.hl7.org/fhir/r3maps.html) states that the maps are subject to ongoing maintenance using the FHIR NPM Package "fhir.versions.r3r4" which is maintained on [GitHub](https://github.com/FHIR/r3r4). This repository/ig will be submitted as a pull request for HL7, see [Zulip discusion](https://chat.fhir.org/#narrow/stream/179166-implementers/topic/Maps.20np.20mpackage.20for.20R3.20to.20R4.3F).

1. Changes between FHIR source github:

* renamed BodyStructure.map to BodySite.map
* renamed CoverageEligibilityRequest.map to EligibilityRequest.map
* renamed CoverageEligibilityResponse.map to EligibilityResponse.map
* renamed MolecularSequence3.map to Sequence.map

2. Parsing Rule names keeps the double quotes for the rule names which is not valid for a rule name: StructureMap/ActivityDefinition3to4: StructureMap.group[3].rule[2].name error id value '"expression"' is not valid. 

[Created a pull request/with a test case for that](https://github.com/hapifhir/org.hl7.fhir.core/pull/49)

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




