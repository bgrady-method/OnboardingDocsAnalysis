# D: Drive Setup

## 1.3. Create D: drive for logging

The local logging expects an available D: drive. The simplest way to do this is:

1. Windows Key+R => diskmgmt.msc => Select C: => Shrink Volume  
   ![][image5]

2. Set the new unused space to D:

## Why D: Drive is Required

Method's logging infrastructure is configured to write logs to the D: drive by default. This includes:

- Application error logs
- Debug information
- Service logs
- Build output logs

## Alternative Approaches

If you cannot create a D: drive, you may need to:
1. Modify logging configuration in various projects
2. Use symbolic links to redirect D: paths to another location
3. Contact the development team for configuration assistance

⚠️ **Recommended:** Creating an actual D: drive partition is the simplest approach

**Next:** [Credentials Setup](../credentials/README.md)
