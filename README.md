# npg_id_generation

An API used to generate product IDs, which are hashes of the JSON representation
of an object.

For different sequencing platforms different sets of identifiers might be used to
fully describe the origin of data. For reasons of efficiency and interobility
between different systems it is sometimes desirable to be able to use a single
identifier, which will be unique not only within data for a single platform,
but also between different platforms.

In the Sanger Institute run ID, lane number and numerical tag index are used
as identifiers for the Illumina platform. Historically, the first algorithm for
generating unique identifiers was implemented in Perl for the Illumina platform,
see [documentation](https://github.com/wtsi-npg/npg_tracking/blob/master/lib/npg_tracking/glossary/composition.pm
).

Later a need to have a similar API for other sequencing platforms arose. This
package implements a Python API. The attributes of objects are sequencing
platform specific. The generator for the PacBio platform is implemented by the
`PacBioEntity` class.

Examles of generating IDs for PacBio data:

```
from npg_id_generation.main import PacBioEntity

# from a JSON string via a class method
test_case = '{"run_name": "MARATHON","well_label": "D1"}'
print(PacBioEntity.parse_raw(test_case, content_type="json").hash_product_id())

# by setting object's attributes
print(PacBioEntity(run_name="MARATHON", well_label="D1").hash_product_id()

# sample-specific indentifier
# for multiple tags a sorted comma-separated list of tagscan be used
print(PacBioEntity(run_name="MARATHON", well_label="D1", tags="AAGTACGT").hash_product_id()
``` 

All generators should conform to a few simple rules:

1. Uniquness of the ID should be guaranteed.
2. The ID should be a 64 characher string.
3. It should be possible to generate an ID from a JSON string.
4. The value of the ID should **not** depend on the order of attributes given
   to the cinstructor or the order of keys used in JSON.
5. The value of the ID should **not** depend on the amount of whitespace in
   the input JSON.
6. The value of the ID should **not** depend on whether the undefined values
   of attributes are explicitly set.

The examples below clarity the rules. Objects `o1` - `o6` should generate the same ID.

```
o1 = PacBioEntity(run_name="r1", well_label="l1")
o2 = PacBioEntity(run_name="r1", well_label="l1", tags = None)
o3 = PacBioEntity(well_label="l1", run_name="r1", )
o4 = PacBioEntity.parse_raw('{"run_name": "r1","well_label": "l1"}', content_type="json")
o5 = PacBioEntity.parse_raw('{"well_label": "l1",  "run_name": "r1"}', content_type="json")
o6 = PacBioEntity.parse_raw('{"well_label": "l1","run_name": "r1", "tags": null}', content_type="json")
```

The algorithm used for generation of identifiers can be replicated in Perl;
on identical input data it gives identical results. However, we cannot guarantee
that this parity will always be maintained in future.
