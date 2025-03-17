**********************************************************************
* Writes any type of Internal Table to an AL11 path, in CSV format.
* This API can be released & used in ABAP Cloud.
**********************************************************************

CLASS zcl_al11_utility DEFINITION
  PUBLIC
  FINAL
  CREATE PUBLIC .

  PUBLIC SECTION.
    CLASS-METHODS add_to_al11
      IMPORTING in_file_data     TYPE REF TO data
                in_file_path     TYPE string
      RETURNING VALUE(out_error) TYPE string.
  PROTECTED SECTION.
  PRIVATE SECTION.
    DATA: file_data TYPE REF TO data.
    DATA: file_path TYPE string.
    METHODS process RETURNING VALUE(error) TYPE string.
    METHODS transform_to_csv EXPORTING csv_data TYPE crmtt_tdd_textascii.
ENDCLASS.


CLASS zcl_al11_utility IMPLEMENTATION.
  METHOD add_to_al11.

    DATA(instance) = NEW zcl_al11_utility(  ).
    instance->file_data = REF #( in_file_data ).
    instance->file_path = in_file_path.
    out_error = instance->process(  ).

  ENDMETHOD.

  METHOD process.

    DATA: dataset_msg TYPE string.

    CLEAR error.

    me->transform_to_csv( IMPORTING csv_data = DATA(csv_data) ).
    IF csv_data IS INITIAL.
      error = 'Empty File'.
      RETURN.
    ENDIF.

    TRY.

        OPEN DATASET me->file_path FOR OUTPUT IN TEXT MODE ENCODING DEFAULT MESSAGE dataset_msg.
        IF sy-subrc = 0.
          LOOP AT csv_data ASSIGNING FIELD-SYMBOL(<record>).
            TRANSFER <record> TO me->file_path.
          ENDLOOP.
        ELSE.
          error = SWITCH #( dataset_msg WHEN space THEN 'Unable to open file' ELSE dataset_msg ).
          RETURN.
        ENDIF.
        CLOSE DATASET me->file_path.

      CATCH: cx_sy_file_open cx_sy_codepage_converter_init cx_sy_conversion_codepage
             cx_sy_file_authority cx_sy_pipes_not_supported cx_sy_too_many_files
             cx_sy_file_io cx_sy_file_open_mode cx_sy_pipe_reopen cx_sy_file_close
             INTO FINAL(exception).
        FINAL(dataset_exp) = exception->get_text( ).
        error = dataset_exp.
    ENDTRY.

  ENDMETHOD.

  METHOD transform_to_csv.

    DATA structdescr TYPE REF TO cl_abap_structdescr.
    DATA csv_head    TYPE string.
    DATA csv_line    TYPE string.

    FIELD-SYMBOLS <file_data> TYPE any.
    FIELD-SYMBOLS <value> TYPE any.

    CLEAR csv_data.
    ASSIGN me->file_data->* TO <file_data>.

    LOOP AT <file_data>->* ASSIGNING FIELD-SYMBOL(<record>).
      structdescr ?= cl_abap_typedescr=>describe_by_data( <record> ).
      DATA(line_components) = structdescr->get_components( ).
      EXIT.
    ENDLOOP.

    LOOP AT <file_data>->* ASSIGNING <record>.

      CLEAR: csv_head, csv_line.
      DATA(index) = sy-tabix.

      LOOP AT line_components ASSIGNING FIELD-SYMBOL(<component>). "#EC CI_NESTED "Needed to prepare CSV

        IF index = 1.
          IF csv_head IS INITIAL.
            csv_head = <component>-name.
          ELSE.
            csv_head = |{ csv_head },{ <component>-name }|.
          ENDIF.
        ENDIF.

        ASSIGN COMPONENT <component>-name OF STRUCTURE <record> TO <value>.
        IF sy-subrc = 0 AND <value> IS ASSIGNED.
          IF csv_line IS INITIAL.
            csv_line = <value>.
          ELSE.
            csv_line = |{ csv_line },{ <value> }|.
          ENDIF.
        ENDIF.

      ENDLOOP.

      IF index = 1.
        csv_data = VALUE #( BASE csv_data ( csv_head ) ).
      ENDIF.

      IF csv_line IS NOT INITIAL.
        csv_data = VALUE #( BASE csv_data ( csv_line ) ).
      ENDIF.
    ENDLOOP.

  ENDMETHOD.

ENDCLASS.
