# Estagio-ALV-GRID-POOL-Editavel

### ALV-GRID-MODULE POOL EDITÁVEL

```abap
REPORT z_editable_alv_grid.


TYPES BEGIN OF y_orders_c.

TYPES prod    TYPE ztable_02_prod2-produto.
TYPES desc    TYPE ztable_02_prod2-desc_produto.
TYPES price   TYPE ztable_02_prod2-preco.
TYPES price2  TYPE ztable_02_prod2-preco.
TYPES celltab TYPE lvc_t_styl.

TYPES END OF y_orders_c.

DATA: lo_grid     TYPE REF TO cl_gui_alv_grid,
      lt_fieldcat TYPE lvc_t_fcat,
      lt_data     TYPE TABLE OF y_orders_c.

DATA: gs_alv_layout TYPE lvc_s_layo.

DATA: e_cell_type   TYPE lvc_s_styl.

DATA: e_orders_c    TYPE y_orders_c.

START-OF-SELECTION.

  CALL SCREEN 100.

*&---------------------------------------------------------------------*
*&      Module  SHOW_ORDERS  OUTPUT
*&---------------------------------------------------------------------*
MODULE show_orders OUTPUT.

  IF NOT lo_grid IS BOUND.

    "CATÁLOGO DOS CAMPOS
    PERFORM f_fieldcat USING lt_data[] CHANGING lt_fieldcat.

    LOOP AT lt_fieldcat  ASSIGNING FIELD-SYMBOL(<fs_fieldcat>).

      CASE <fs_fieldcat>-fieldname.
        WHEN 'PROD'.
          <fs_fieldcat>-coltext = 'PRODUTO'.

        WHEN 'DSEC'.
          <fs_fieldcat>-coltext = 'DESCRIÇÃO DO PRODUTO'.

        WHEN 'PRICE'.
          <fs_fieldcat>-coltext = 'PREÇO ORIGINAL'.

        WHEN 'PRICE2'.
          <fs_fieldcat>-edit = 'X'. "SET FIELD TO EDITABLE
          <fs_fieldcat>-coltext = 'NOVO PREÇO'.
      ENDCASE.
    ENDLOOP.

      "CONFIGURAÇÃO DE LAYOUT
      CLEAR gs_alv_layout.
      MOVE abap_true TO: gs_alv_layout-cwidth_opt,
                         gs_alv_layout-zebra,
                         gs_alv_layout-col_opt.
      gs_alv_layout-stylefname = 'CELLTAB'.


      "COLOCAR DADOS
      e_orders_c-prod = '1'.
      e_orders_c-desc = 'PRESUNTO'.
      e_orders_c-price = '200'.
      e_orders_c-price2 = '2'.
      APPEND e_orders_c TO lt_data.

      CLEAR e_orders_c-prod.
      e_orders_c-prod = '2'.
      e_orders_c-desc = 'QUEIJO'.
      e_orders_c-price = '400'.
      e_orders_c-price2 = '4'.
      APPEND e_orders_c TO lt_data.

      CLEAR e_orders_c-prod.
      e_orders_c-prod = '3'.
      e_orders_c-desc = 'PÃO'.
      e_orders_c-price = '300'.
      e_orders_c-price2 = '3'.
      APPEND e_orders_c TO lt_data.

      CLEAR e_orders_c-prod.
      e_orders_c-prod = '4'.
      e_orders_c-desc = 'REFRIGERANTE'.
      e_orders_c-price = '500'.
      e_orders_c-price2 = '5'.
      APPEND e_orders_c TO lt_data.

      CLEAR e_orders_c-prod.
      e_orders_c-prod = '5'.
      e_orders_c-desc = 'CHOCOLATE'.
      e_orders_c-price = '700'.
      e_orders_c-price2 = '7'.
      APPEND e_orders_c TO lt_data.

      CREATE OBJECT lo_grid
        EXPORTING
          i_parent = cl_gui_container=>screen0.

      CALL METHOD  lo_grid->set_table_for_first_display
        EXPORTING
          i_structure_name = 'LT_DATA'
          is_layout        = gs_alv_layout
        CHANGING
          it_outtab        = lt_data
          it_fieldcatalog  = lt_fieldcat.

      CALL METHOD lo_grid->register_edit_event
        EXPORTING
          i_event_id = cl_gui_alv_grid=>mc_evt_enter
        EXCEPTIONS
          error  = 1
          OTHERS = 2.
      IF sy-subrc <> 0.
      MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
        WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
    ENDIF.

      ELSE.
        PERFORM f_update_grid USING lo_grid.
   ENDIF.

ENDMODULE.


*&---------------------------------------------------------------------*
*&      Module  STATUS_0100  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE status_0100 OUTPUT.
  SET PF-STATUS 'STATUS100'.
*  SET TITLEBAR 'xxx'.
ENDMODULE.


*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_0100  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE user_command_0100 INPUT.

  CASE sy-ucomm.
    WHEN 'BACK' OR 'EXIT' OR 'CANCEL'.
      LEAVE TO SCREEN '0'.
    WHEN 'SWITCH'.
      PERFORM f_switch_edit_mode.
    WHEN OTHERS.
   ENDCASE.

ENDMODULE.

*&---------------------------------------------------------------------*
*&      Form  F_FIELDCAT
*&---------------------------------------------------------------------*

FORM f_fieldcat  USING pt_table TYPE ANY TABLE CHANGING pt_fieldcat TYPE lvc_t_fcat.

  DATA:
    lr_tabdescr TYPE REF TO cl_abap_structdescr,
    lr_data TYPE REF TO data,
    lt_dfies  TYPE ddfields,
    ls_dfies TYPE dfies,
    ls_fieldcat TYPE lvc_s_fcat.

  CLEAR pt_fieldcat.

  " ?= is used to type cast an object reference of an inherited class to an object of the super class from which it is derived
  CREATE DATA lr_data LIKE LINE OF pt_table.
  lr_tabdescr ?= cl_abap_structdescr=>describe_by_data_ref( lr_data ).
  lt_dfies = cl_salv_data_descr=>read_structdescr( lr_tabdescr ).

  LOOP AT lt_dfies INTO ls_dfies.
    CLEAR ls_fieldcat.
    MOVE-CORRESPONDING ls_dfies TO ls_fieldcat.
    APPEND ls_fieldcat TO pt_fieldcat.
  ENDLOOP.

ENDFORM.


*&---------------------------------------------------------------------*
*&      Form  F_SWITCH_EDIT_MODE
*&---------------------------------------------------------------------*

FORM f_switch_edit_mode .

  IF lo_grid->is_ready_for_input( ) EQ 0.

  CALL METHOD lo_grid->set_ready_for_input
    EXPORTING
      i_ready_for_input = 1.

  ELSE.
    CALL METHOD lo_grid->set_ready_for_input
      EXPORTING
       i_ready_for_input = 0.
  ENDIF.

ENDFORM.


*&---------------------------------------------------------------------*
*&      Form  F_UPDATE_GRID
*&---------------------------------------------------------------------*
FORM f_update_grid USING p_grid TYPE REF TO cl_gui_alv_grid.

  DATA vl_stable TYPE lvc_s_stbl.

  CHECK p_grid IS BOUND.

  CLEAR vl_stable.
  MOVE abap_true TO:
    vl_stable-col,
    vl_stable-row.

  CALL METHOD p_grid->refresh_table_display
    EXPORTING
      is_stable = vl_stable
    EXCEPTIONS
      OTHERS    = 99.

  IF sy-subrc NE 0.
    RETURN.
  ENDIF.

ENDFORM.
```
