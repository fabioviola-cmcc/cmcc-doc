# Monitoring WRF

## Step 0 -- Before 8.45 am

On the telegram channel **WRF Messages** we can receive one of the following messages (`$DP`: production date as `YYYYMMDD`):

- ":tada: Il 16° dei venti di oggi (20201110) è andato! :tada:" -- **SUCCESS**
- ":scream: Il 16° dei venti di oggi (20201111) ci ha lasciati! La cosa è grave :weary:" -- **FAILURE**

In case of FAILURE see **Recover A**

## Recover A -- Before 10 am

1. Open [Luigi](http://athena01.cmcc.scc:58082/static/visualiser/index.html#) and check the process `WindToENI`, to get information from the red bullets.
2. Connect to Athena:
```bash
ssh gofs@athena01.cmcc.scc
```
3. Check that files `${area}_1.16_XXX.grib` are present:
```bash
area="Angola Arctic Australia Britain Caribbean Caspian.Sea China.Sea Guinea Gulf.Mex Med.Central Med.East Med.West Mozambique Red.Sea"
for i in ${area}; do  echo $i;  ls /work/tessa_gpfs2/wrf/output/${i}_1.16_*.grib |wc -l; done
```

Two options:
- Files are 157 for every region -- **SUCCESS**
- One or more regions have less than 157 files -- **FAILURE**

In case of FAILURE see **Recover B**

## Recover B -- Before 10 am

Check which regions are incomplete. 
- If **Red sea** or **Arctic** are the incomplete regions, we can immediately send the email below.
- Else, the problem *may be* on our side. We check that we have enough space by logging in as `das@okeanos.cmcc.scc` and issuing `df -h`. At least 3.5 TB should be available.

Now we can write an email to `eni_atm@cmcc.it`:

```
Dear all,
this mail is to inform you that into the folder of WRF today's production (/work/tessa gpfs2/wrf/output/) there were some missing outputs:

– list of missing outputs

Could you please check the encountered issues and provide a description of them for internal tracking purposes?
Thanks a lot.
Best Regards
```

## Recover C -- Before 11 am

When files are missing, we have to provide our backup data. So:

1. Open [Luigi](http://athena01.cmcc.scc:58082/static/visualiser/index.html#) and go to Workers, then kill the process SubmitWindToEni (generally shown at the end of the page)
2. Connect to Athena and kill job EniWnd16:
```bash
$ ssh gofs@athena01.cmcc.scc
$ bkill -J EniWnd16
```
3. Check that backup files are present (`DP` is set to yesterday):
```bash
DP=YYYYMMDD
area="Angola Arctic Australia Britain Caribbean Caspian.Sea South.China Guinea Gulf.Mex Med.Central Med.East Med.West Mozambique Red.Sea"
for i in ${area}; do echo ${i};ls  /work/tessa_gpfs2/archive/rolling/raw/atmos/NCEP/GFS025/1.0forecast/6h/${DP}/backup/${i}_1.16_*.grib|wc -l;done
```
**NOTE**: this region list is different from the one used in step *Recover A*!


After the last command, we have two options:
- Files are 121 for every region. -- **SUCCESS**
- One or more regions have less than 121 files. -- **FAILURE**

In case of SUCCESS:
```bash
$ cd /users/home/gofs/wrf
$ nohup runEniWindBKP.sh &
```

In case of FAILURE see **Recover D**

## Recover D -- Before 11 am

If backup files are also missing:

```bash
$ ssh das@okeanos.cmcc.scc
$ sh /users/home/tessa_gpfs1/das/NCEP/post-proc/script/copy/process_ENI_good.sh
```

Then go back to **Recover C, point 3**

## Recover E -- Before 11.30 am

If files are still missing, disable the upload to ENI.

1. Connect to zeus and open crontab:
```bash
$ ssh gofs@zeus01.cmcc.scc
$ EDITOR=nano crontab -e
```
2. Comment the following line:
```bash
30 11 * * * /users_home/opa/gofs/gofs_op/publication/md5_check.sh > /users_home/opa/gofs/gofs_op/publication/upload.log 2>&1
```

**NOTE:** remember to uncomment this line after 11.30!





